---
title: Solución de problemas de Azure Percept Vision y de los módulos de visión
description: Vea sugerencias para solucionar algunos de los problemas más comunes que se encuentran en las experiencias de creación de prototipos de inteligencia artificial de visión.
author: mimcco
ms.author: mimcco
ms.service: azure-percept
ms.topic: how-to
ms.date: 03/29/2021
ms.custom: template-how-to
ms.openlocfilehash: 7befc9648e696e80e0469d5e91173786354574d8
ms.sourcegitcommit: b35c7f3e7f0e30d337db382abb7c11a69723997e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109685754"
---
# <a name="vision-solution-troubleshooting"></a>Solución de problemas de las soluciones de visión

Consulte la siguiente guía para obtener información sobre cómo solucionar problemas de las soluciones de visión sin código en Azure Percept Studio.

## <a name="delete-a-vision-project"></a>Eliminación de un proyecto de visión

1. Ir a https://www.customvision.ai/projects.

1. Mantenga el mouse sobre el proyecto que desea eliminar y haga clic en el icono de la papelera para eliminar el proyecto.

    :::image type="content" source="./media/vision-solution-troubleshooting/vision-delete-project.png" alt-text="Página Proyectos de Custom Vision con el icono Eliminar resaltado.":::

## <a name="check-which-modules-are-on-a-device"></a>Comprobación de qué módulos hay en un dispositivo

1. Vaya a [Azure Portal](https://portal.azure.com/?feature.canmodifystamps=true&Microsoft_Azure_Iothub=aduprod&microsoft_azure_marketplace_ItemHideKey=Microsoft_Azure_ADUHidden#home).

1. Haga clic en el icono de **IoT Hub**.

    :::image type="content" source="./media/vision-solution-troubleshooting/vision-iot-hub-2-inline.png" alt-text="Página principal de Azure Portal con el icono de IoT Hub resaltado." lightbox= "./media/vision-solution-troubleshooting/vision-iot-hub-2.png":::

1. Seleccione el icono de IoT Hub al que está conectado el dispositivo de destino.

    :::image type="content" source="./media/vision-solution-troubleshooting/vision-iot-hub.png" alt-text="Lista de instancias de IoT Hub.":::

1. Seleccione **IoT Edge** y haga clic en el dispositivo en la pestaña **Id. de dispositivo**.

    :::image type="content" source="./media/vision-solution-troubleshooting/vision-iot-edge.png" alt-text="Página principal de IoT Edge.":::

1. Los módulos del dispositivo aparecerán en la pestaña **Módulos**.

    :::image type="content" source="./media/vision-solution-troubleshooting/vision-device-modules-inline.png" alt-text="Página de IoT Edge del dispositivo seleccionado que muestra el contenido de la pestaña Módulos." lightbox= "./media/vision-solution-troubleshooting/vision-device-modules.png":::

## <a name="delete-a-device"></a>Eliminar un dispositivo

1. Vaya a [Azure Portal](https://portal.azure.com/?feature.canmodifystamps=true&Microsoft_Azure_Iothub=aduprod&microsoft_azure_marketplace_ItemHideKey=Microsoft_Azure_ADUHidden#home).

1. Haga clic en el icono de **IoT Hub**.

1. Seleccione el icono de IoT Hub al que está conectado el dispositivo de destino.

1. Seleccione **IoT Edge** y active la casilla situada junto al identificador del dispositivo de destino. Haga clic en el icono de la papelera para eliminar el dispositivo.

    :::image type="content" source="./media/vision-solution-troubleshooting/vision-delete-device.png" alt-text="Icono Eliminar resaltado en la página principal de IoT Edge.":::

## <a name="eye-module-troubleshooting-tips"></a>Sugerencias para la solución de problemas del módulo de visión

### <a name="check-the-runtime-status-of-azureeyemodule"></a>Comprobar el estado de runtime de azureeyemodule

Si hay un problema con **WebStreamModule**, asegúrese de que **azureeyemodule**, que realiza la inferencia de modelos de visión, se esté ejecutando. Para comprobar el estado del entorno de ejecución, vaya a [Azure Portal](https://portal.azure.com/?feature.canmodifystamps=true&Microsoft_Azure_Iothub=aduprod&microsoft_azure_marketplace_ItemHideKey=Microsoft_Azure_ADUHidden#home) y a **Todos los recursos** ->  **\<your IoT hub>**  -> **IoT Edge** ->  **\<your device ID>** . Haga clic en la pestaña **Módulos** para ver el estado del entorno de ejecución de todos los módulos instalados.

:::image type="content" source="./media/vision-solution-troubleshooting/over-the-air-iot-edge-device-page-inline.png" alt-text="Pantalla de estado del entorno de ejecución del módulo del dispositivo." lightbox= "./media/vision-solution-troubleshooting/over-the-air-iot-edge-device-page.png":::

Si el estado del entorno de ejecución de **azureeyemodule** no aparece como **En ejecución**, haga clic en **Establecer módulos** -> **azureeyemodule**. En la página **Configuración del módulo**, establezca **Estado deseado** como **En ejecución** y haga clic en **Actualizar**.

 :::image type="content" source="./media/vision-solution-troubleshooting/firmware-desired-status-stopped.png" alt-text="Pantalla de configuración de opciones del módulo.":::

### <a name="update-telemetryintervalneuralnetworkms"></a>Actualizar TelemetryIntervalNeuralNetworkMs

Si se produce el siguiente error de limitación de recuento, será necesario actualizar el valor de TelemetryIntervalNeuralNetworkMs en la configuración gemela del módulo azureeyemodule.

|Mensaje de error|
|------|
|El número total de mensajes en IotHub 'XXXXXXXXX' superó la cuota asignada. Número máximo de mensajes permitidos: ' 8000', número de mensajes actual: 'XXXX'. Las operaciones de envío y recepción se bloquean para este centro hasta el siguiente día UTC. Considere la posibilidad de aumentar las unidades de este centro para aumentar la cuota.|

TelemetryIntervalNeuralNetworkMs determina con qué frecuencia se envían los mensajes (en milisegundos) desde la red neuronal. Las suscripciones de Azure tienen un número limitado de mensajes por día, según el nivel de suscripción. Si está bloqueado debido a que ha enviado demasiados mensajes, auméntelo a un número mayor. 12000 (lo que significa que una vez cada 12 segundos) le proporcionará un buen ciclo 7200 mensajes al día, que se encuentra en el límite de 8000 mensajes para la suscripción gratuita.

Para actualizar el valor TelemetryIntervalNeuralNetworkMs, siga los siguientes pasos:

1. Inicie sesión en el [Azure Portal](https://ms.portal.azure.com/?feature.canmodifystamps=true&Microsoft_Azure_Iothub=aduprod#home) y Abra **Todos los recursos**.

1. En la página **Todos los recursos** , haga clic en el nombre del IoT Hub que se aprovisionó en el DevKit durante la experiencia de instalación.

1. En el lado izquierdo de la página IoT Hub, haga clic en **IoT Edge** en **Administración automática de dispositivos**. En la página dispositivos IoT Edge, busque el identificador de dispositivo de su DevKit. Haga clic en el identificador de dispositivo de su DevKit para abrir su página de dispositivo de IoT Edge.

1. Seleccione **azureeyemodule** en la pestaña **Módulos**.

1. En la página azureeyemodule, abra el **Módulo de identidad gemela**.

    :::image type="content" source="./media/vision-solution-troubleshooting/module-page-inline.png" alt-text="Captura de pantalla de la página de módulos." lightbox= "./media/vision-solution-troubleshooting/module-page.png":::

1. Desplácese hacia **propiedades**. Tenga en cuenta que las propiedades "Running" y "Logging" no están activas en este momento.

    :::image type="content" source="./media/vision-solution-troubleshooting/module-identity-twin-inline.png" alt-text="Captura de pantalla de las propiedades de dispositivos gemelos." lightbox= "./media/vision-solution-troubleshooting/module-identity-twin.png":::

1. Actualice el valor de **TelemetryIntervalNeuralNetworkMs** como desee y haga clic en el icono **Guardar**.

## <a name="view-device-rtsp-video-stream"></a>Visualización de la secuencia de vídeo RTSP del dispositivo

Vea la secuencia de vídeo RTSP de su dispositivo en [Azure Percept Studio](./how-to-view-video-stream.md) o [VLC Media Player](https://www.videolan.org/vlc/index.html).

Para abrir la secuencia RTSP en VLC Media Player, vaya a **Media (Medios)**  -> **Open network stream (Abrir secuencia de red)**  -> **rtsp://[dirección IP del dispositivo]:8554/result**.

## <a name="next-steps"></a>Pasos siguientes

Consulte la [guía general de solución de problemas](./troubleshoot-dev-kit.md) para más información sobre la solución de problemas de Azure Percept DK.