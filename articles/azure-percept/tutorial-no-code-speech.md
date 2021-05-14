---
title: Creación de un asistente para voz con Azure Percept DK y Azure Percept Audio
description: Aprenda a crear e implementar una solución de voz sin código en Azure Percept DK.
author: philmea
ms.author: philmea
ms.service: azure-percept
ms.topic: tutorial
ms.date: 02/17/2021
ms.custom: template-how-to
ms.openlocfilehash: 441d0b4b76e0c543bd82c5546e0ca89250e418e7
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108748176"
---
# <a name="create-a-voice-assistant-with-azure-percept-dk-and-azure-percept-audio"></a>Creación de un asistente para voz con Azure Percept DK y Azure Percept Audio

En este tutorial, creará un asistente para voz a partir de una plantilla para usarlo con Azure Percept DK y Azure Percept Audio. La demostración del asistente para voz se ejecuta en [Azure Percept Studio](https://go.microsoft.com/fwlink/?linkid=2135819) y contiene una selección de objetos virtuales controlados por voz. Para controlar un objeto, diga la palabra clave, que es una palabra o frase corta que reactiva el dispositivo, seguido de un comando. Cada plantilla responde a un conjunto de comandos específicos.

Esta guía le mostrará el proceso de configuración de los dispositivos, la creación de un asistente para voz y los recursos necesarios de los [servicios de voz](../cognitive-services/speech-service/overview.md), la prueba del asistente, la configuración de la palabra clave y la creación de palabras clave personalizadas.

## <a name="prerequisites"></a>Prerrequisitos

- Kit de desarrollo Azure Percept DK
- Azure Percept Audio
- Altavoces o auriculares que se pueden conectar a un conector de audio de 3,5 mm (opcional)
- [Suscripción de Azure](https://azure.microsoft.com/free/)
- [Experiencia de instalación de Azure Percept DK](./quickstart-percept-dk-set-up.md): ya ha conectado el kit de desarrollo a una red Wi-Fi, creado una instancia de IOT Hub y conectado el kit a esta instancia.
- [Instalación de Azure Percept Audio](./quickstart-percept-audio-setup.md)


## <a name="create-a-voice-assistant-using-an-available-template"></a>Creación de un asistente para voz con una plantilla disponible

1. Vaya a [Azure Percept Studio](https://go.microsoft.com/fwlink/?linkid=2135819).

1. Abra la pestaña **Demos & tutorials** (Demostraciones y tutoriales).

    :::image type="content" source="./media/tutorial-no-code-speech/portal-overview.png" alt-text="Captura de pantalla de la página principal de Azure Portal.":::

1. Haga clic en **Try out voice assistant templates** (Probar plantillas del asistente para voz) en **Speech tutorials and demos** (Tutoriales y demostraciones de voz). Con esto, se abrirá una ventana en el lado derecho de la pantalla.

1. Haga lo siguiente en la ventana:

    1. En el menú desplegable **IoT Hub**, seleccione el centro de IoT al que está conectado el kit de desarrollo.

    1. En el menú desplegable **Device** (Dispositivo), seleccione el kit de desarrollo.

    1. Seleccione una de las plantillas disponibles del asistente para voz.

    1. Haga clic en la casilla **I agree to terms & conditions for this project** (Acepto los términos y condiciones de este proyecto).

    1. Haga clic en **Crear**.

    :::image type="content" source="./media/tutorial-no-code-speech/template-creation.png" alt-text="Captura de pantalla de creación de una plantilla del asistente para voz.":::

1. Después de hacer clic en **Create** (Crear), el portal abre otra ventana para crear el recurso del tema de voz. Haga lo siguiente en la ventana:

    1. Seleccione la suscripción de Azure en el campo **Suscription** (Suscripción).

    1. Seleccione el grupo de recursos que prefiera en el menú desplegable **Resource group** (Grupo de recursos). Si desea crear un nuevo grupo de recursos para usarlo con el asistente para voz, haga clic en **Create** (Crear) en el menú desplegable y siga las indicaciones.

    1. En **Application prefix** (Prefijo de la aplicación), escriba un nombre. Este será el prefijo del proyecto y el nombre del comando personalizado.

    1. En **Región**, seleccione la región en la que se van a implementar los recursos.

    1. En **LUIS prediction pricing tier** (Plan de tarifa de predicción de LUIS), seleccione **Standard** (Estándar) (el nivel gratuito no admite solicitudes de voz).

    1. Haga clic en el botón **Crear**. Los recursos de la aplicación del asistente para voz se implementarán en la suscripción.

        > [!WARNING]
        > **NO** cierre la ventana hasta que el portal termine de implementar el recurso. Si cierra la ventana anticipadamente puede producir un comportamiento inesperado del asistente para voz. Una vez implementado el recurso, se mostrará la demostración.

    :::image type="content" source="./media/tutorial-no-code-speech/resource-group.png" alt-text="Captura de pantalla de la ventana de selección de la suscripción y el grupo de recursos.":::

## <a name="test-out-your-voice-assistant"></a>Prueba del asistente para voz

Para interactuar con el asistente para voz, indique la palabra clave seguida de un comando. Cuando el módulo de sistema de escucha reconoce la palabra clave, el dispositivo emite el sonido de una campanilla (que se puede oír si hay un altavoz o unos auriculares conectados) y los indicadores LED parpadean en azul. Los LED parpadean rápidamente en azul mientras se procesa el comando. La respuesta del asistente para voz al comando se imprimirá en texto en la ventana de demostración y se emitirá de forma audible a través de los altavoces o auriculares. La palabra clave predeterminada (que aparece junto a **Palabra clave personalizada**) está establecida en "Computer" (Equipo) y cada plantilla tiene un conjunto de comandos compatibles que le permiten interactuar con objetos virtuales en la ventana de demostración. Por ejemplo, si usa la demostración para hostelería o la de atención sanitaria, diga "Equipo, enciende la televisión" para encender la televisión virtual.

:::image type="content" source="./media/tutorial-no-code-speech/hospitality-demo.png" alt-text="Captura de pantalla de la ventana de demostración para hostelería.":::

### <a name="hospitality-and-healthcare-demo-commands"></a>Comandos de demostración para hostelería y atención sanitaria

Las demostraciones de atención sanitaria y hostelería tienen televisores virtuales, luces, persianas y termostatos con los que puede interactuar. Se admiten los siguientes comandos (y otras variantes):

* "Apaga las luces."
* "Enciende/Apaga la televisión."
* "Enciende/Apaga el aire acondicionado."
* "Abre/Cierra las persianas."
* "Establece la temperatura en X grados." (Donde X es la temperatura deseada, por ejemplo, 24.)

:::image type="content" source="./media/tutorial-no-code-speech/healthcare-demo.png" alt-text="Captura de pantalla de la ventana de demostración para atención sanitaria.":::

### <a name="automotive-demo-commands"></a>Comandos de demostración para automoción

La demo para automoción dispone de un asiento calefactado, un eliminador de escarcha y un termostato con los que puede interactuar. Se admiten los siguientes comandos (y otras variantes):

* "Apagar/Encender el eliminador de escarcha."
* "Apagar/Encender el asiento calefactado."
* "Establece la temperatura en X grados." (Donde X es la temperatura deseada, por ejemplo, 24.)
* "Aumentar/Disminuir la temperatura en Y grados."


:::image type="content" source="./media/tutorial-no-code-speech/auto-demo.png" alt-text="Captura de pantalla de la ventana de demostración para automoción.":::

### <a name="inventory-demo-commands"></a>Comandos de demostración para inventario

La demostración de inventario tiene una selección de cajas azules, amarillas y verdes con las que interactuar junto con una aplicación de inventario virtual. Se admiten los siguientes comandos (y otras variantes):

* "Agregar/Quitar X cajas." (X es el número de cajas, por ejemplo, 4).
* "Pedir/Enviar X cajas."
* "¿Cuántas cajas hay en existencias?"
* "Contar cajas Y." (Y es el color de las cajas, por ejemplo, amarillo).
* "Enviar todo lo que hay en existencias."


:::image type="content" source="./media/tutorial-no-code-speech/inventory-demo.png" alt-text="Captura de pantalla de la ventana de demostración para inventario.":::

## <a name="configure-your-keyword"></a>Configuración de la palabra clave

Puede personalizar la palabra clave para la aplicación del asistente para voz.

1. Haga clic en **Cambiar** junto a **Palabra clave personalizada** en la página de demostración.

1. Seleccione una de las palabras clave disponibles. Podrá elegir entre una selección de palabras clave de ejemplo y cualquier palabra clave personalizada que haya creado.

1. Haga clic en **Save**(Guardar).

### <a name="create-a-custom-keyword"></a>Creación de una palabra clave personalizada

Puede crear su propia palabra clave para la aplicación de voz. El entrenamiento de la palabra clave personalizada puede completarse en unos minutos.

1. Haga clic en **+ Crear palabra clave personalizada** cerca de la parte superior de la ventana de demostración. 

1. Escriba la palabra clave que desee, que puede ser una sola palabra o una frase corta.

1. Seleccione el **recurso de voz** (está incluido en una lista junto a **Comando personalizado** en la ventana de demostración y contiene el prefijo de la aplicación).

1. Haga clic en **Save**(Guardar). 

## <a name="create-a-custom-command"></a>Creación de un comando personalizado

El portal también proporciona una funcionalidad para crear comandos personalizados con recursos de voz existentes. "Comando personalizado" hace referencia a la propia aplicación del asistente para voz, no a un comando específico dentro de la aplicación ya existente. Al crear un comando personalizado, está creando un nuevo proyecto de voz, que debe seguir desarrollando en [Speech Studio](https://speech.microsoft.com/).

Para crear un nuevo comando personalizado desde la ventana de demostración, haga clic en **+ Create Custom Command** (+ Crear comando personalizado) en la parte superior de la página y haga lo siguiente:

1. Escriba un nombre para el comando personalizado.

1. Escriba una descripción del proyecto (opcional).

1. Seleccione su idioma preferido.

1. Seleccione el recurso de voz.

1. Seleccione el recurso de LUIS.

1. Seleccione el recurso de creación de LUIS o cree uno.

1. Haga clic en **Crear**.

:::image type="content" source="./media/tutorial-no-code-speech/custom-commands.png" alt-text="Captura de pantalla de la ventana de creación de comandos personalizados.":::

Una vez que haya creado un comando personalizado, debe ir a [Speech Studio](https://speech.microsoft.com/) para su posterior desarrollo. Si abre Speech Studio y no ve el comando personalizado en la lista, siga estos pasos:

1. En el panel de menú de la izquierda de Azure Percept Studio, haga clic en **Speech** (Voz) en **AI Projects** (Proyectos de IA).

1. Seleccione la pestaña **Commands** (Comandos).

    :::image type="content" source="./media/tutorial-no-code-speech/ai-projects.png" alt-text="Captura de pantalla de la lista de comandos personalizados disponibles para editar.":::

1. Seleccione el comando personalizado que desea desarrollar. Esto abrirá el proyecto en Speech Studio.

    :::image type="content" source="./media/tutorial-no-code-speech/speech-studio.png" alt-text="Captura de pantalla de la pantalla principal de Speech Studio.":::

Para obtener más información sobre el desarrollo de comandos personalizados, consulte la [documentación del servicio Voz](../cognitive-services/speech-service/custom-commands.md).

## <a name="troubleshooting"></a>Solución de problemas

### <a name="voice-assistant-was-created-but-does-not-respond-to-commands"></a>Se ha creado el asistente para voz pero no responde a los comandos

Compruebe las luces LED en la placa del intermediador:

* Tres luces azules fijas indican que el asistente para voz está listo y esperando la palabra clave.
* Si el indicador LED del centro (L02) está en blanco, el kit de desarrollo ha completado la inicialización y debe configurarse con una palabra clave.
* Si el LED del centro (L02) parpadea en blanco, el módulo de sistema de audio todavía no ha completado la inicialización. La operación puede tardar varios minutos en completarse.

Para más información acerca de los indicadores LED, consulte el [artículo sobre indicadores LED](./audio-button-led-behavior.md).

### <a name="voice-assistant-does-not-respond-to-a-custom-keyword-created-in-speech-studio"></a>El asistente para voz no responde a una palabra clave personalizada creada en Speech Studio.

Esto puede ocurrir si el módulo de voz no está actualizado. Siga estos pasos para actualizar el módulo de voz a la versión más reciente:

1. Haga clic en **Devices** (Dispositivos) en el panel de menú de la izquierda de la página principal de Azure Percept Studio.

1. Busque y seleccione el dispositivo.

    :::image type="content" source="./media/tutorial-no-code-speech/devices.png" alt-text="Captura de pantalla de la lista de dispositivos en Azure Percept Studio.":::

1. En la ventana del dispositivo, seleccione la pestaña **Speech** (Voz).

1. Compruebe la versión del módulo de voz. Si hay disponible una actualización, aparecerá un botón **Update** (Actualizar) junto al número de versión.

1. Haga clic en **Update** (Actualizar) para implementar la actualización del módulo de voz. Por lo general, el proceso de actualización tarda 2-3 minutos en completarse.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando haya terminado de trabajar con la aplicación del asistente para voz, siga estos pasos para eliminar los recursos de voz que implementó durante este tutorial:

1. En [Azure Portal](https://portal.azure.com), seleccione **Grupos de recursos** en el panel de menú de la izquierda o escríbalo en la barra de búsqueda.

    :::image type="content" source="./media/tutorial-no-code-speech/azure-portal.png" alt-text="Captura de pantalla de la página principal de Azure Portal que muestra el panel de menú izquierdo y los grupos de recursos.":::

1. Seleccione el grupo de recursos que necesite.

1. Seleccione los seis recursos que contienen el prefijo de la aplicación y haga clic en el icono **Eliminar** en el panel de menú superior.
\
    :::image type="content" source="./media/tutorial-no-code-speech/select-resources.png" alt-text="Captura de pantalla de los recursos de voz seleccionados para su eliminación.":::

1. Para confirmar la eliminación, escriba **sí** en el cuadro de confirmación, compruebe que ha seleccionado los recursos correctos y haga clic en **Eliminar**.

    :::image type="content" source="./media/tutorial-no-code-speech/delete-confirmation.png" alt-text="Captura de pantalla de la ventana de confirmación de eliminación.":::

> [!WARNING]
> Se eliminarán todas las palabras clave personalizadas creadas con los recursos de voz que va a eliminar y la demostración del asistente para voz dejará de funcionar.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha creado una solución de voz sin código, intente crear una [solución de visión sin código](./tutorial-nocode-vision.md) para Azure Percept DK.