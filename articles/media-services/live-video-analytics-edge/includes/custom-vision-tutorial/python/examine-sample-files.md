---
author: russell-cooks
ms.service: media-services
ms.topic: include
ms.date: 10/05/2020
ms.author: russellcooks
ms.openlocfilehash: 5dd823c618a0d49bef29c473e6c293762b859149
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "94358295"
---
1. En Visual Studio Code, vaya a src/edge. Verá el archivo .env que ha creado junto con algunos archivos de plantilla de implementación.

    La plantilla de implementación hace referencia al manifiesto de implementación del dispositivo perimetral con algunos valores de marcador de posición. El archivo .env contiene los valores de esas variables.
1. A continuación, vaya a la carpeta src/cloud-to-device-console-app. Aquí podrá ver el archivo appsettings.json que creó junto con algunos otros archivos:

   * operations.json: este archivo muestra las distintas operaciones que desea que ejecute el programa.
   * main.py: este es el código del programa de ejemplo, que hace lo siguiente:
    
        * Carga la configuración de la aplicación.
        * Invoca Live Video Analytics en los métodos directos del módulo IoT Edge para crear la topología, crear una instancia del grafo y activar el grafo.
        * Se pone en pausa para que pueda examinar la salida del grafo en la ventana **TERMINAL** y los eventos enviados a IoT Hub en la ventana **SALIDA**.
        * Desactiva la instancia del grafo, la elimina y elimina la topología de grafos.
