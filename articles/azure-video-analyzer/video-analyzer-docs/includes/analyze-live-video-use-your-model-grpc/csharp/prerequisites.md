---
author: Juliako
ms.service: azure-video-analyzer
ms.topic: include
ms.date: 04/07/2021
ms.author: juliako
ms.openlocfilehash: b65fea06b967bc5ccf552276c33b8523d1118e51
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110371912"
---
* Una cuenta de Azure que incluya una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), en caso de que aún no lo haya hecho.

    > [!NOTE]
    > Necesitará una suscripción a Azure con al menos un rol de colaborador. Si no tiene los permisos adecuados, póngase en contacto con el administrador de la cuenta para que se los conceda.
* [Visual Studio Code](https://code.visualstudio.com/) con las siguientes extensiones:
    * [Herramientas de Azure IoT](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)
    
    > [!TIP]
    > Al instalar Azure IoT Tools, es posible que se le pida que instale Docker. Si lo desea, ignore esta petición.    
    * [C#](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp)
* [SDK de .NET Core 3.1](https://dotnet.microsoft.com/download/dotnet-core/3.1). 

## <a name="set-up-azure-resources"></a>Configuración de los recursos de Azure

[![Implementación en Azure](https://aka.ms/deploytoazurebutton)](https://aka.ms/ava-click-to-deploy)  
[!INCLUDE [resources](../../../includes/common-includes/azure-resources.md)]

## <a name="overview"></a>Información general

> [!div class="mx-imgBorder"]
> :::image type="content" source="./../../../media/analyze-live-video-use-your-model-grpc/overview.png" alt-text="Introducción a gRPC":::
 
En este diagrama se muestra cómo fluyen las señales en este inicio rápido. Un [módulo perimetral](https://github.com/Azure/video-analyzer/tree/main/edge-modules/sources/rtspsim-live555) simula una cámara IP que hospeda un servidor de protocolo RTSP. Un [nodo de origen RTSP](./../../../pipeline.md#rtsp-source) extrae la fuente de vídeo de este servidor y envía fotogramas de vídeo al nodo del [procesador de detección del movimiento](./../../../pipeline.md#motion-detection-processor). Este procesador detectará el movimiento y, cuando se detecte, enviará fotogramas de vídeo al nodo del [procesador de extensión gRPC](./../../../pipeline.md#grpc-extension-processor).

El nodo de extensión gRPC desempeña el rol de un servidor proxy. Convierte los fotogramas de vídeo en el tipo de imagen especificado. Luego, retransmite la imagen sobre gRPC a otro módulo perimetral que ejecuta un modelo de IA detrás de un punto de conexión gRPC en una [memoria compartida](https://en.wikipedia.org/wiki/Shared_memory). En este ejemplo, el módulo perimetral se crea mediante el modelo [YOLOv3](https://github.com/Azure/video-analyzer/tree/main/edge-modules/extensions/yolo/yolov3), que puede detectar muchos tipos de objetos. El nodo del procesador de extensión gRPC recopila los resultados de la detección y publica los eventos en el nodo del [receptor de IoT Hub](./../../../pipeline.md#iot-hub-message-sink). que posteriormente los envía a [IoT Edge Hub](../../../../../iot-fundamentals/iot-glossary.md?view=iotedge-2020-11&preserve-view=true#iot-edge-hub).

En este inicio rápido realizará lo siguiente:

1. Creará e implementará la canalización.
1. Interpretará los resultados.
1. Limpieza de recursos.

## <a name="set-up-your-development-environment"></a>Configurado su entorno de desarrollo
[!INCLUDE [setup development environment](./../../../includes/set-up-dev-environment/csharp/csharp-set-up-dev-env.md)]