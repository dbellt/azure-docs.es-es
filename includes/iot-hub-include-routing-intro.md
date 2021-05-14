---
title: archivo de inclusión
description: archivo de inclusión
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: include
ms.date: 03/05/2019
ms.author: robinsh
ms.custom: include file
ms.openlocfilehash: f2d91e9405c17afe44a05deedc8bb0f0c40377d7
ms.sourcegitcommit: 43be2ce9bf6d1186795609c99b6b8f6bb4676f47
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2021
ms.locfileid: "108278315"
---
[El enrutamiento de mensajes](../articles/iot-hub/iot-hub-devguide-messages-d2c.md) permite enviar datos de telemetría desde dispositivos de IoT a puntos de conexión integrados compatibles con el centro de eventos o puntos de conexión personalizados como Blob Storage, colas de Service Bus, temas de Service Bus y Event Hubs. Para configurar el enrutamiento de mensajes personalizado, cree [consultas de enrutamiento](../articles/iot-hub/iot-hub-devguide-routing-query-syntax.md) para personalizar la ruta que coincide con una condición determinada. Una vez configurado, los datos entrantes se enrutan automáticamente a los puntos de conexión mediante IoT Hub. Si un mensaje no coincide con ninguna de las consultas de enrutamientos definidas, se enruta al punto de conexión predeterminado.

En este tutorial de dos partes, aprenderá a configurar y usar las consultas de enrutamiento personalizadas con IoT Hub. Enrutará los mensajes desde un dispositivo IoT a uno de los múltiples puntos de conexión, entre los que se incluyen Blob Storage y una cola de Service Bus. Una aplicación lógica recogerá los mensajes que se envían a la cola de Service Bus, que se enviarán por correo electrónico. Los mensajes que no tienen definido el enrutamiento de mensajes personalizado se envían al punto de conexión predeterminado y, a continuación, Azure Stream Analytics los recoge y se muestran en una visualización de Power BI.

Para completar las dos partes de este tutorial, realice las siguientes tareas:

**Parte I: Creación de recursos, configuración del enrutamiento de mensajes**
> [!div class="checklist"]
> * Cree los recursos: una instancia de IoT Hub, una cuenta de almacenamiento, una cola de Service Bus y un dispositivo simulado. Esto puede hacerse mediante Azure Portal, la CLI de Azure, una plantilla de Azure Resource Manager o Azure PowerShell.
> * Configure los puntos de conexión y las rutas de los mensajes en IoT Hub para la cuenta de almacenamiento y la cola de Service Bus.

**Parte II: Envío de mensajes al centro, visualización de los resultados del enrutamiento**
> [!div class="checklist"]
> * Crear una aplicación lógica que se desencadena y se envía un correo electrónico cuando se agrega un mensaje a la cola de Service Bus.
> * Descargar y ejecutar una aplicación que simule que un dispositivo IoT envía mensajes al centro para las diferentes opciones de enrutamiento.
> * Crear una visualización de Power BI para los datos enviados al punto de conexión predeterminado.
> * Vea los resultados...
> * ... en la cola de Service Bus y en los mensajes de correo electrónico.
> * ... en la cuenta de almacenamiento.
> * ... en la visualización de Power BI.

## <a name="prerequisites"></a>Requisitos previos

* Para la primera parte de este tutorial:
  - Debe tener una suscripción de Azure. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

* Para la segunda parte de este tutorial:
  - Debe haber completado la parte I de este tutorial y seguir disponiendo de los recursos.
  - Instale [Visual Studio](https://www.visualstudio.com/).
  - Debe tener acceso a una cuenta de Power BI para realizar el análisis del flujo del punto de conexión predeterminado. ([pruebe Power BI de manera gratuita](https://app.powerbi.com/signupredirect?pbi_source=web)).
  - Debe tener una cuenta profesional o educativa para enviar notificaciones por correo electrónico.
  - Asegúrese de que el puerto 8883 está abierto en el firewall. En el ejemplo de este tutorial se usa el protocolo MQTT, que se comunica mediante el puerto 8883. Este puerto puede estar bloqueado en algunos entornos de red corporativos y educativos. Para más información y para saber cómo solucionar este problema, consulte el artículo sobre la [conexión a IoT Hub (MQTT)](../articles/iot-hub/iot-hub-mqtt-support.md#connecting-to-iot-hub).

[!INCLUDE [cloud-shell-try-it.md](cloud-shell-try-it.md)]
