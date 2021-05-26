---
author: Juliako
ms.service: azure-video-analyzer
ms.topic: include
ms.date: 04/30/2021
ms.author: juliako
ms.openlocfilehash: 559f4eccbf1eb9bd41471937b3f1d6e846812b40
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110384134"
---
1. En Visual Studio Code, vaya a src/edge. Verá el archivo .env que ha creado junto con algunos archivos de plantilla de implementación.

   La plantilla de implementación hace referencia al manifiesto de implementación del dispositivo perimetral con algunos valores de marcador de posición. El archivo .env contiene los valores de esas variables.
2. A continuación, vaya a la carpeta src/cloud-to-device-console-app. Aquí podrá ver el archivo appsettings.json que creó junto con algunos otros archivos:

   - operations.json: este archivo muestra las distintas operaciones que desea que ejecute el programa.
   - main.py: este es el código del programa de ejemplo, que hace lo siguiente:
     - Carga la configuración de la aplicación.
     - Invoca los métodos directos del módulo de Azure Video Analyzer para crear una topología, crear una instancia de la canalización y activarla.
     - Se pone en pausa para que pueda examinar la salida de la canalización en la ventana **TERMINAL** y los eventos enviados a IoT Hub en la ventana **SALIDA**.
     - Desactiva la canalización en directo y la elimina, y elimina la topología.
