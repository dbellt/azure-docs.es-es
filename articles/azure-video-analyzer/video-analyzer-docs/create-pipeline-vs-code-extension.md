---
title: Extensión de Visual Studio Code de Azure Video Analyzer
description: Este inicio rápido le guiará por los pasos necesarios para empezar a usar la extensión Visual Studio Code para Azure Video Analyzer.
ms.service: azure-video-analyzer
ms.topic: quickstart
ms.date: 04/30/2021
ms.openlocfilehash: bfbb73172cc137be5c8ed20333b2efc468ec8af1
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388556"
---
# <a name="quickstart-azure-video-analyzer-visual-studio-code-extension"></a>Inicio rápido: Uso de la extensión de Visual Studio Code para Azure Video Analyzer

Este inicio rápido está diseñado para mostrarle cómo configurar y conectar la extensión de Visual Studio Code para Video Analyzer al módulo perimetral de Video Analyzer e implementar una topología de canalización de ejemplo.  Usará la misma canalización que en el inicio rápido de introducción.  

Después de completar los pasos de la configuración, podrá ejecutar la secuencia de vídeo en directo simulada mediante una canalización activa que detecta e informa de todos los movimientos de esa secuencia. El siguiente diagrama representa gráficamente esa canalización.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/get-started-detect-motion-emit-events/motion-detection.svg" alt-text="Detección de movimiento":::
 
 ## <a name="prerequisites"></a>Requisitos previos
 
* Una cuenta de Azure que incluya una suscripción activa. [Cree una cuenta](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) gratuita, en caso de que aún no lo haya hecho.

* [Visual Studio Code](https://code.visualstudio.com/) con las siguientes extensiones:
    * [Video Analyzer](https://go.microsoft.com/fwlink/?linkid=2163332)

* Si no ha completado el inicio rápido [Introducción: Azure Video Analyzer](./get-started-detect-motion-emit-events.md), asegúrese de [configurar los recursos de Azure](#set-up-azure-resources).    

## <a name="set-up-azure-resources"></a>Configuración de los recursos de Azure

[![Implementación en Azure](https://aka.ms/deploytoazurebutton)](https://aka.ms/ava-click-to-deploy)

[!INCLUDE [resources](./includes/common-includes/azure-resources.md)]

## <a name="connect-the-azure-video-analyzer-visual-studio-code-extension-to-your-iot-hub"></a>Conexión de la extensión de Visual Studio Code para Azure Video Analyzer a su IoT Hub

Para conectar la extensión al módulo perimetral, primero debe recuperar la cadena de conexión. Para ello, siga estos pasos.

1.  Vaya a [Azure Portal](https://portal.azure.com) y seleccione su instancia de IoT Hub.
1.  A la izquierda, en `Settings`, seleccione `Shared access policies`.
1.  Seleccione el nombre de la directiva `iothubowner`.
1.  En la ventana de la derecha, copie `Primary connection string`.

Ahora que tiene la cadena de conexión, los pasos siguientes conectarán la extensión al módulo perimetral.

1.  En Visual Studio Code, seleccione el icono de `Azure Video Analyzer` a la izquierda.
1.  Haga clic en el botón `Enter Connection String`.
1.  En la parte superior, pegue la cadena de conexión del portal.
1.  Seleccione el dispositivo: el valor predeterminado es `avasample-iot-edge-device`.
1.  Seleccione el módulo de Video Analyzer: el valor predeterminado es `avaedge`.

A la izquierda, ahora verá el dispositivo conectado con el módulo subyacente.  De forma predeterminada, no hay ninguna topología de canalización implementada.

## <a name="deploy-a-topology-and-live-pipeline"></a>Implementación de una topología y una canalización activa

Las topologías de canalización son el bloque de creación básico que Video Analyzer usa para definir cómo se produce el trabajo.  Puede obtener más información sobre las [topologías de canalización aquí](./pipeline.md).  En esta sección, implementará una topología de canalización que es una plantilla y, a continuación, creará una instancia de la topología o canalización activa. La canalización activa está conectada a la secuencia de vídeo real.

1.  A la izquierda, en `Modules`, haga clic con el botón derecho en `Pipeline topologies` y seleccione "Create pipeline topology" (Crear topología de canalización).
1.  En la parte superior, en `Try sample topologies`, en `Motion Detection`, seleccione `Publish motion events to IoT Hub`.  Cuando se le solicite, haga clic en `Proceed`.
1.  Haga clic en `Save` en la parte superior derecha.

Ahora debería ver una entrada en la lista `Pipeline topologies` de la izquierda con la etiqueta `MotionDetection`.  Se trata de una topología de canalización en la que algunos de los parámetros se definen como variables que se pueden alimentar al crear una canalización activa.  A continuación, crearemos una canalización activa.

1.  A la izquierda, en `Pipeline topologies`, haga clic con el botón derecho en `MotionDetection` y seleccione `Create live pipeline`.
1.  En `Live pipeline name`, escriba `mdpipeline1`.
1.  En la sección `Parameters`:
    - En "rtspPassword", escriba "testuser".
    - En "rtspUrl", escriba "rtsp://rtspsim:554/media/camera-300s.mkv".
    - En "rtspUserName", escriba "testpassword".
1.  En la parte superior derecha, haga clic en "Guardar y activar".

Esto le permite obtener una topología inicial implementada y una canalización activa en funcionamiento en el dispositivo perimetral.  Si tiene instalada la extensión de Azure IoT Hub desde el inicio rápido Introducción, puede supervisar el punto de conexión integrado en la extensión de Visual Studio Code para Azure IoT Hub para supervisar esto como se indica en la sección [Observación de resultados](./get-started-detect-motion-emit-events.md#observe-results).

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [prerequisites](./includes/common-includes/clean-up-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre las distintas funciones de la [extensión de Visual Studio Code para Azure Video Analytics](./visual-studio-code-extension.md).
