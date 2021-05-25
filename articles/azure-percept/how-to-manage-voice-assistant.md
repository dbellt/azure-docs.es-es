---
title: Configuración de una aplicación de asistente para voz en Azure Percept Studio
description: Configuración de una aplicación de asistente para voz en Azure Percept Studio
author: mimcco
ms.author: mimcco
ms.service: azure-percept
ms.topic: how-to
ms.date: 02/15/2021
ms.custom: template-how-to
ms.openlocfilehash: 764ca28ec6e66d63ffeb98399f654b14eb9b6715
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108754296"
---
# <a name="managing-your-voice-assistant"></a>Administración del asistente para voz

En este artículo se describe cómo configurar la palabra clave y los comandos de la aplicación del asistente para voz en [Azure Percept Studio](https://go.microsoft.com/fwlink/?linkid=2135819). Para obtener instrucciones sobre la configuración de la palabra clave en IoT Hub en lugar del portal, consulte [Configuración de la aplicación de asistente para voz mediante Azure IoT Hub](./how-to-configure-voice-assistant.md).

Si aún no ha creado una aplicación de asistente para voz, consulte [Creación de un asistente para voz con Azure Percept DK y Azure Percept Audio](./tutorial-no-code-speech.md).

## <a name="keyword-configuration"></a>Configuración de palabras clave

Una palabra clave es una palabra o frase corta utilizada para activar un asistente para voz. Por ejemplo, "Hola Cortana" es la palabra clave para el asistente Cortana. La activación por voz permite que los usuarios empiecen a interactuar con su producto sin tener que usar las manos, simplemente diciendo la palabra clave. Como el producto siempre espera escuchar la palabra clave, todo el audio se procesa de manera local en el dispositivo hasta que se produce una detección para asegurarse de que los datos del usuario sigan siendo lo más privados posible.

### <a name="configuration-within-the-voice-assistant-demo-window"></a>Configuración en la ventana de demostración del asistente para voz

1. Seleccione **Change** (Cambiar) junto a **Custom Keyword** (Palabra clave personalizada) en la página de demostración.

    :::image type="content" source="./media/manage-voice-assistant/hospitality-demo.png" alt-text="Captura de pantalla de la ventana de demostración para hostelería.":::

    Si no tiene abierta la página de demostración, vaya a la página del dispositivo (véase a continuación) y seleccione **Test your voice assistant** (Probar el asistente para voz) en **Actions** (Acciones) para acceder a la demostración.

1. Seleccione una de las palabras clave disponibles y seleccione **Save** (Guardar) para aplicar los cambios.

1. Las tres luces LED del dispositivo Azure Percept Audio cambiarán a azul brillante (sin parpadeo) cuando se complete la configuración y el asistente para voz esté listo para usarse.

### <a name="configuration-within-the-device-page"></a>Configuración dentro de la página del dispositivo

1. En la página de información general de [Azure Percept Studio](https://go.microsoft.com/fwlink/?linkid=2135819), seleccione **Devices** (Dispositivos) en el panel del menú de la izquierda.

    :::image type="content" source="./media/manage-voice-assistant/portal-overview-devices.png" alt-text="Captura de pantalla de la página de información general de Azure Percept Studio con la opción Dispositivos resaltada.":::

1. Seleccione el dispositivo en el que se implementó la aplicación de asistente para voz.

1. Abra la pestaña **Speech** (Voz).

    :::image type="content" source="./media/manage-voice-assistant/device-page.png" alt-text="Captura de pantalla de la página de dispositivo perimetral con la pestaña Voz resaltada.":::

1. Seleccione **Change** (Cambiar) junto a **Keyword** (Palabra clave).

    :::image type="content" source="./media/manage-voice-assistant/change-keyword-device.png" alt-text="Captura de pantalla de las acciones de la solución de voz disponibles.":::

1. Seleccione una de las palabras clave disponibles y seleccione **Save** (Guardar) para aplicar los cambios.

1. Las tres luces LED del dispositivo Azure Percept Audio cambiarán a azul brillante (sin parpadeo) cuando se complete la configuración y el asistente para voz esté listo para usarse.

## <a name="create-a-custom-keyword"></a>Creación de una palabra clave personalizada

Con [Speech Studio](https://speech.microsoft.com/), puede crear una palabra clave personalizada para el asistente para voz. Se tardan hasta 30 minutos en entrenar un modelo básico de palabras clave personalizadas.

Siga la [documentación de Speech Studio](../cognitive-services/speech-service/custom-keyword-basics.md) para obtener instrucciones sobre cómo crear una palabra clave personalizada. Una vez configurada, la nueva palabra clave estará disponible en el portal de Project Santa Cruz para su uso con la aplicación de asistente para voz.

## <a name="commands-configuration"></a>Configuración de comandos

Los comandos personalizados facilitan la creación de aplicaciones de comandos de voz enriquecidas y optimizadas para las experiencias de interacción que priorizan la voz. Los comandos personalizados son más adecuados para escenarios de finalización de tareas o de comando y control.

### <a name="configuration-within-the-voice-assistant-demo-window"></a>Configuración en la ventana de demostración del asistente para voz

1. Seleccione **Change** (cambiar) junto a **Custom Command** (Comando personalizado) en la página de demostración. Si no tiene abierta la página de demostración, vaya a la página del dispositivo (véase a continuación) y seleccione **Test your voice assistant** (Probar el asistente para voz) en **Actions** (Acciones) para acceder a la demostración.

1. Seleccione uno de los comandos personalizados disponibles y seleccione **Save** (Guardar) para aplicar los cambios.

### <a name="configuration-within-the-device-page"></a>Configuración dentro de la página del dispositivo

1. En la página de información general de [Azure Percept Studio](https://go.microsoft.com/fwlink/?linkid=2135819), seleccione **Devices** (Dispositivos) en el panel del menú de la izquierda.

1. Seleccione el dispositivo en el que se implementó la aplicación de asistente para voz.

1. Abra la pestaña **Speech** (Voz).

1. Seleccione **Change** (Cambiar) junto a **Command** (Comando).

1. Seleccione uno de los comandos personalizados disponibles y seleccione **Save** (Guardar) para aplicar los cambios.

## <a name="create-custom-commands"></a>Creación de comandos personalizados

Con [Speech Studio](https://speech.microsoft.com/), puede crear comandos personalizados para que los ejecute el asistente para voz.

Siga la [documentación de Speech Studio](../cognitive-services/speech-service/quickstart-custom-commands-application.md) para obtener instrucciones sobre cómo crear comandos personalizados. Una vez configurados, los nuevos comandos estarán disponibles en Azure Percept Studio para su uso con la aplicación de asistente para voz.

## <a name="next-steps"></a>Pasos siguientes

Después de compilar una aplicación de asistente para voz, pruebe a desarrollar una [solución de visión sin código](./tutorial-nocode-vision.md) con Azure Percept DK.