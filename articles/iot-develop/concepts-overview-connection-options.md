---
title: Más información sobre las opciones de conexión para desarrolladores de dispositivos de Azure IoT
description: Obtenga más información sobre las opciones y herramientas de conexión que usan habitualmente los desarrolladores de dispositivos de Azure IoT.
author: timlt
ms.author: timlt
ms.service: iot-develop
ms.topic: conceptual
ms.date: 02/11/2021
ms.openlocfilehash: 8669919192b1e6394043842d7d23f8829ec7c71e
ms.sourcegitcommit: 272351402a140422205ff50b59f80d3c6758f6f6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/17/2021
ms.locfileid: "107589557"
---
# <a name="overview-connection-options-for-azure-iot-device-developers"></a>Introducción sobre las opciones de conexión para desarrolladores de dispositivos de Azure IoT
Como desarrollador que trabaja con dispositivos, tiene varias opciones para conectar y administrar dispositivos de Azure IoT. En este artículo se muestra una visión general de las opciones y herramientas que se usan con más frecuencia para ayudarle a conectar y administrar dispositivos.

Al evaluar las opciones de conexión de Azure IoT para sus dispositivos, resulta útil comparar los siguientes elementos:
- Plataformas de aplicaciones de dispositivos de Azure IoT
- Herramientas para la conexión y administración de dispositivos

## <a name="application-platforms-iot-central-and-iot-hub"></a>Plataformas de aplicaciones: IoT Central e IoT Hub
Azure IoT incluye dos servicios que son plataformas para aplicaciones en la nube habilitadas para dispositivos: Azure IoT Central y Azure IoT Hub. Puede usar cualquiera de ellos para hospedar una aplicación de IoT y conectar dispositivos.
- [IoT Central](../iot-central/core/overview-iot-central.md) está diseñado para reducir la complejidad y el costo de trabajar con soluciones de IoT. Es una aplicación de software como servicio (SaaS) que proporciona una plataforma completa para el hospedaje de aplicaciones de IoT. Puede usar la interfaz de usuario web de IoT Central para optimizar todo el ciclo de vida de la creación y administración de aplicaciones de IoT. La interfaz de usuario web simplifica las tareas de creación de aplicaciones y la conexión y administración de hasta varios millones de dispositivos. IoT Central usa IoT Hub para crear y administrar aplicaciones, pero mantiene los detalles claros para el usuario. En general, IoT Central proporciona una menor complejidad y esfuerzo de desarrollo, y la administración de dispositivos simplificada a través de la interfaz de usuario web.
- [IoT Hub](../iot-hub/about-iot-hub.md) funciona como un centro de conectividad de mensajes central para la comunicación bidireccional entre aplicaciones de IoT y los dispositivos conectados. Se trata de una aplicación de plataforma como servicio (PaaS) que también proporciona una plataforma para hospedar aplicaciones de IoT. Al igual que IoT Central, se puede escalar para admitir millones de dispositivos. En general, IoT Hub ofrece un mayor control y personalización sobre el diseño de la aplicación, así como más opciones de herramientas de desarrollo para trabajar con el servicio, a costa de un aumento en la complejidad del desarrollo y la administración.

## <a name="tools-to-connect-and-manage-devices"></a>Herramientas para conectar y administrar dispositivos
Después de seleccionar IoT Hub o IoT Central para hospedar su aplicación de IoT, tiene varias opciones de herramientas de desarrollo. Puede usar estas herramientas para conectarse a la plataforma de aplicaciones de IoT seleccionada y para crear y administrar aplicaciones y dispositivos. En la tabla siguiente se resumen las opciones de herramientas habituales. 

> [!NOTE]
> Además de las siguientes herramientas, puede crear y administrar aplicaciones de IoT mediante programación con las API REST, los SDK de Azure o las plantillas de Azure Resource Manager. Puede obtener más información en la documentación de servicio de [IoT Hub](../iot-hub/about-iot-hub.md) e [IoT Central](../iot-central/core/overview-iot-central.md).

|Herramienta  |Admite la plataforma de IoT &nbsp; &nbsp; &nbsp; &nbsp; |Documentación  |Descripción  |
|---------|---------|---------|---------|
|Interfaz de usuario web central     | Central | [Inicio rápido de IoT Central](../iot-central/core/quick-deploy-iot-central.md) | Portal basado en un explorador para IoT Central. |
|Azure Portal     | Hub, Central      | [Creación de un centro de IoT con Azure Portal](../iot-hub/iot-hub-create-through-portal.md), [Administración de IoT Central desde Azure Portal](../iot-central/core/howto-manage-iot-central-from-portal.md)| Portal basado en un explorador para IoT Hub y dispositivos. También funciona con otros recursos de Azure incluido IoT Central. |
|Azure IoT Explorer     | Hub | [Azure IoT Explorer](https://github.com/Azure/azure-iot-explorer#azure-iot-explorer-preview) | No se pueden crear centros de IoT. Se conecta a un centro de IoT existente para administrar los dispositivos. Se usa a menudo con la CLI o Azure Portal.|
|Azure CLI     | Hub, Central          | [Creación de un centro de IoT con la CLI](../iot-hub/iot-hub-create-using-cli.md), [Administración de IoT Central desde la CLI de Azure](../iot-central/core/howto-manage-iot-central-from-cli.md) | Interfaz de la línea de comandos para crear y administrar aplicaciones de IoT. |
|Azure PowerShell     | Hub, Central   | [Creación de un centro de IoT con PowerShell](../iot-hub/iot-hub-create-using-powershell.md), [Administración de IoT Central desde Azure PowerShell](../iot-central/core/howto-manage-iot-central-from-powershell.md) | Interfaz de PowerShell para crear y administrar aplicaciones de IoT |
|Azure IoT Tools para VS Code  | Hub | [Creación de un centro de IoT con herramientas de Visual Studio Code](../iot-hub/iot-hub-create-use-iot-toolkit.md) | Extensión de Visual Studio Code para aplicaciones de IoT Hub. |

## <a name="next-steps"></a>Pasos siguientes
Para más información sobre las opciones para conectar dispositivos a Azure IoT, explore los siguientes inicios rápidos:
- IoT Central: [Creación de una aplicación de Azure IoT Central](../iot-central/core/quick-deploy-iot-central.md)
- IoT Hub: [Enviar telemetría desde un dispositivo a IoT Hub y supervisarlo con la CLI de Azure](../iot-hub/quickstart-send-telemetry-cli.md)