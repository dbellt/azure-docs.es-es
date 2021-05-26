---
author: fvneerden
ms.service: azure-video-analyzer
ms.topic: include
ms.date: 03/18/2021
ms.author: faneerde
ms.openlocfilehash: cc93611bf0fc1b6c86ac937e09d296adf1e609f5
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110486360"
---
Para ejecutar el código de ejemplo, siga estos pasos:

1. En Visual Studio Code, abra la pestaña **Extensiones** (o presione Ctrl + Mayús + X) y busque Azure IoT Hub.
1. Haga clic con el botón derecho y seleccione la **Configuración de la extensión**.

   > [!div class="mx-imgBorder"]
   > :::image type="content" source="../../../media/vscode-common-screenshots/extension-settings.png" alt-text="Configuración de la extensión":::

1. Busque y habilite "Show Verbose Message" (Mostrar mensaje detallado).

   > [!div class="mx-imgBorder"]
   > :::image type="content" source="../../../media/vscode-common-screenshots/verbose-message.png" alt-text="Show Verbose Message"::: (Mostrar mensaje detallado)

1. En Visual Studio Code, vaya a _src/cloud-to-device-console-app/operations.json_.
1. En el nodo **pipelineTopologySet**, asegúrese de que ve el siguiente valor:

   ```
   "pipelineTopologyUrl" : "https://raw.githubusercontent.com/Azure/video-analyzer/main/pipelines/live/topologies/motion-detection/topology.json"
   ```

1. En los nodos `livePipelineSet` y `livePipelineDelete`, asegúrese de que el valor de **topologyName** coincide con el valor de la propiedad **name** en la topología de la canalización:

   `"topologyName" : "MotionDetection"`

1. Para iniciar una sesión de depuración, seleccione la tecla F5. La ventana **TERMINAL** mostrará algunos mensajes.
1. El archivo _operations.json_ se inicia con llamadas a `pipelineTopologyList` y `livePipelineList`. Si limpió los recursos al finalizar los inicios rápidos anteriores, este proceso devolverá listas vacías.

   ```
   -----------------------  Request: pipelineTopologyList  --------------------------------------------------

   {
   "@apiVersion": "1.0"
   }

   ---------------  Response: pipelineTopologyList - Status: 200  ---------------

   {
     "value": []
   }

   --------------------------------------------------------------------------

   ```

   La ventana **TERMINAL** muestra el siguiente conjunto de llamadas al método directo:

   - Una llamada a `pipelineTopologySet` que utiliza el nodo pipelineTopologyUrl anterior.
   - Una llamada a `livePipelineSet` que usa el cuerpo siguiente:

   ```json
   {
     "@apiVersion": "1.0",
     "name": "Sample-Pipeline-1",
     "properties": {
       "topologyName": "MotionDetection",
       "description": "Sample pipeline description",
       "parameters": [
         {
           "name": "rtspUrl",
           "value": "rtsp://rtspsim:554/media/camera-300s.mkv"
         },
         {
           "name": "rtspUserName",
           "value": "testuser"
         },
         {
           "name": "rtspPassword",
           "value": "testpassword"
         }
       ]
     }
   }
   ```

   - Una llamada a `livePipelineActivate` que activa la canalización en directo y el flujo de vídeo.
   - Una segunda llamada a `livePipelineList` que muestra que la canalización en directo está en estado de ejecución.

1. La salida de la ventana **TERMINAL** se pone en pausa en `Press Enter to continue`. No seleccione Entrar todavía. Desplácese hacia arriba para ver las cargas de la respuesta JSON para los métodos directos que ha invocado.
1. Cambie a la ventana **SALIDA** de Visual Studio Code. Verá los mensajes que el módulo perimetral de Video Analyzer envía al centro de IoT. En la siguiente sección de este inicio rápido se analizan estos mensajes.
1. Prosigue la canalización en directo y se imprimen los resultados. El simulador RTSP sigue recorriendo el vídeo de origen. Para detener la canalización en directo, vuelva a la ventana **TERMINAL** y seleccione Entrar.

   La siguiente serie de llamadas limpia los recursos:

   - Una llamada a `livePipelineDeactivate` desactiva la canalización.
   - Una llamada a `livePipelineDelete` elimina la canalización.
   - Una llamada a `pipelineTopologyDelete` elimina la topología.
   - Una llamada final a `pipelineTopologyList` muestra que la lista está vacía.
