---
title: Configuración de la aplicación de asistente para voz mediante Azure IoT Hub
description: Configuración de la aplicación de asistente para voz mediante Azure IoT Hub
author: mimcco
ms.author: mimcco
ms.service: azure-percept
ms.topic: how-to
ms.date: 02/15/2021
ms.custom: template-how-to
ms.openlocfilehash: 514bee115ee0e01910fe726ee8ae378352f89d99
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108754368"
---
# <a name="configure-voice-assistant-application-using-azure-iot-hub"></a>Configuración de la aplicación de asistente para voz mediante Azure IoT Hub

En este artículo se describe cómo configurar la aplicación de asistente para voz mediante IoT Hub. Para obtener un tutorial paso a paso del proceso de creación de un asistente para voz, vea [Compilación de un asistente para voz sin código con Azure Percept Studio y Azure Percept Audio](./tutorial-no-code-speech.md).

## <a name="update-your-voice-assistant-configuration"></a>Actualización de la configuración del asistente para voz

1. Abra [Azure Portal](https://portal.azure.com) y escriba **IoT Hub** en la barra de búsqueda. Seleccione el icono para abrir la página de IoT Hub.

1. En la página de IoT Hub, seleccione la instancia de IoT Hub en la que se aprovisionó el dispositivo.

1. Seleccione **IoT Edge** en **Administración automática de dispositivos** en el menú de navegación izquierdo para ver todos los dispositivos conectados a su instancia de IoT Hub.

1. Seleccione el dispositivo en el que se implementó la aplicación de asistente para voz.

1. Seleccione **Set modules** (Establecer módulos).

    :::image type="content" source="./media/manage-voice-assistant-using-iot-hub/set-modules.png" alt-text="Captura de pantalla de la página de dispositivos con la opción Establecer módulos resaltada.":::

1. Compruebe que la siguiente entrada está presente en la sección **Credenciales de Container Registry**. Agregue credenciales, si fuera necesario.

    |NOMBRE|Dirección|Nombre de usuario|Contraseña|
    |----|-------|--------|--------|
    |azureedgedevices|azureedgedevices.azurecr.io|devkitprivatepreviewpull|

1. En la sección **Módulos de IoT Edge**, seleccione **azureearspeechclientmodule**.

    :::image type="content" source="./media/manage-voice-assistant-using-iot-hub/modules.png" alt-text="Captura de pantalla que muestra la lista de todos los módulos de IoT Edge del dispositivo.":::

1. Seleccione la pestaña **Configuración del módulo**. Compruebe la siguiente configuración:

    URI de imagen|Directiva de reinicio|Estado deseado
    ---------|--------------|--------------
    mcr.microsoft.com/azureedgedevices/azureearspeechclientmodule: preload-devkit|Siempre|en ejecución

    Si la configuración no coincide, edítela y seleccione **Actualizar**.

1. Seleccione la pestaña **Variables de entorno**. Compruebe que no haya variables de entorno definidas.

1. Seleccione la pestaña **Configuración de módulos gemelos**. Actualice la sección **speechConfigs** como se indica a continuación:

    ```
    "speechConfigs": {
        "appId": "<Application id for custom command project>",
        "key": "<Speech Resource key for custom command project>",
        "region": "<Region for the speech service>",
        "keywordModelUrl": "https://aedsamples.blob.core.windows.net/speech/keyword-tables/computer.table",
        "keyword": "computer"
    }
    ```

    > [!NOTE]
    > La palabra clave usada anteriormente es una palabra clave predeterminada disponible públicamente. Si desea usar la suya propia, puede agregar su propia palabra clave personalizada cargando un archivo de tabla creado en el almacenamiento de blobs. El almacenamiento de blobs se debe configurar con acceso de contenedor anónimo o con acceso de blobs anónimo.

## <a name="how-to-find-out-appid-key-and-region"></a>Obtención del identificador de la aplicación, la clave y la región

Para obtener el **identificador de la aplicación**, la **clave** y la **región**, vaya a [Speech Studio](https://speech.microsoft.com/):

1. Inicie sesión y seleccione el recurso de voz adecuado.
1. En la página principal de **Speech Studio**, seleccione **Comandos personalizados** en **Asistentes para voz**.
1. Seleccione el proyecto de destino.

    :::image type="content" source="./media/manage-voice-assistant-using-iot-hub/project.png" alt-text="Captura de pantalla de la página del proyecto en Speech Studio.":::

1. Seleccione **Configuración** en el panel de menús izquierdo.
1. El **identificador de la aplicación** y la **clave** se encuentran en la pestaña de configuración **General**.

    :::image type="content" source="./media/manage-voice-assistant-using-iot-hub/general-settings.png" alt-text="Captura de pantalla de la configuración general del proyecto de voz.":::

1. Para encontrar la **región**, abra la pestaña **Recursos de LUIS** en la configuración. La selección de **Recurso de creación** contendrá la información de la región.

    :::image type="content" source="./media/manage-voice-assistant-using-iot-hub/luis-resources.png" alt-text="Captura de pantalla de los recursos de LUIS del proyecto de voz.":::

1. Después de escribir la información de **speechConfigs**, seleccione **Actualizar**.

1. Seleccione la pestaña **Rutas** en la parte superior de la página **Establecer módulos**. Asegúrese de tener una ruta con el siguiente valor:

    ```
    FROM /messages/modules/azureearspeechclientmodule/outputs/* INTO $upstream
    ```

    Agregue la ruta si no existe.

1. Seleccione **Revisar + crear**.

1. Seleccione **Crear**.


## <a name="next-steps"></a>Pasos siguientes

Después de actualizar la configuración del asistente para voz, vuelva a la demostración de Azure Percept Studio para interactuar con la aplicación.

