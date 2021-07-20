---
title: 'Inicio rápido: Incorporación de Defender para IoT a una solución basada en agente'
description: En este inicio rápido aprenderá a incorporar y habilitar el servicio de seguridad Defender para IoT en Azure IoT Hub.
ms.topic: quickstart
ms.date: 1/20/2021
ms.openlocfilehash: ff3dd6ec3002802ea5f698560b1ead02aee25201
ms.sourcegitcommit: a038863c0a99dfda16133bcb08b172b6b4c86db8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/29/2021
ms.locfileid: "113011495"
---
# <a name="quickstart-onboard-defender-for-iot-to-an-agent-based-solution"></a>Inicio rápido: Incorporación de Defender para IoT a una solución basada en agente

En este artículo se explica cómo habilitar el servicio Defender para IoT en la instancia existente de IoT Hub. Si actualmente no tiene una instancia de IoT Hub, consulte [Creación de una instancia de IoT Hub mediante Azure Portal](../../iot-hub/iot-hub-create-through-portal.md) para comenzar.

Puede administrar la seguridad de IoT mediante IoT Hub en Defender para IoT. El portal de administración ubicado en IoT Hub le permite hacer lo siguiente: 

- Administración de la seguridad de IoT Hub.

- Administración básica de la seguridad de un dispositivo IoT sin necesidad de instalar un agente basado en la telemetría de IoT Hub. 

- Administración avanzada para la seguridad de un dispositivo IoT basado en el microagente.

> [!NOTE]
> Actualmente, Defender para IoT solo admite el nivel Estándar de las instancias de IoT Hub.

## <a name="prerequisites"></a>Requisitos previos

Ninguno

## <a name="onboard-defender-for-iot-to-an-iot-hub"></a>Incorporación de Defender para IoT a un centro de IoT

Para todos los nuevos centros de IoT, Defender para IoT se establece en **Activado** de forma predeterminada. Puede comprobar que Defender para IoT está **Activado** durante el proceso de creación de IoT Hub.

Para comprobar si está **Activado**:

1. Acceda a Azure Portal.

1. Seleccione **IoT Hub** en la lista de servicios de Azure.

1. Seleccione **Crear**.

    :::image type="content" source="media/quickstart-onboard-iot-hub/create-iot-hub.png" alt-text="Seleccione el botón Crear en la barra de herramientas superior." lightbox="media/quickstart-onboard-iot-hub/create-iot-hub-expanded.png":::

1. Seleccione la pestaña **Administración** y compruebe que el botón de alternancia de **Defender para IoT** está establecido en **Activado**.

    :::image type="content" source="media/quickstart-onboard-iot-hub/management-tab.png" alt-text="Asegúrese de que el botón de alternancia de Defender para IoT esté establecido en Activado.":::

## <a name="onboard-defender-for-iot-to-an-existing-iot-hub"></a>Incorporación de Defender para IoT a una instancia de IoT Hub existente

Puede incorporar Defender para IoT a un centro de IoT existente, en el que después podrá supervisar la administración de identidades de dispositivos y los patrones de comunicación del dispositivo a la nube y viceversa.

Para incorporar Defender para IoT a un centro de IoT existente:

1. Vaya a su instancia de IoT Hub. 

1. Seleccione el centro de IoT donde va a incorporar el servicio.

1. En la sección **Seguridad**, seleccione cualquier opción.

1. Haga clic en  **Secure your IoT solution** (Proteger la solución de IoT) y tcomplete el formulario de incorporación. 

    :::image type="content" source="media/quickstart-onboard-iot-hub/secure-your-iot-solution.png" alt-text="Seleccione el botón de protección de la solución de IoT para proteger la solución.":::

El botón **Secure your IoT solution** (Proteger la solución de IoT) solo aparecerá si el centro de IoT aún no se ha incorporado, o si durante la incorporación dejó la opción Defender para IoT como **Desactivada**.

:::image type="content" source="media/quickstart-onboard-iot-hub/toggle-is-off.png" alt-text="Si el botón de alternancia estaba establecido en Desactivado durante la incorporación.":::

## <a name="next-steps"></a>Pasos siguientes

Pase al siguiente artículo para configurar una solución...

> [!div class="nextstepaction"]
> [Creación de un módulo gemelo del microagente de Defender para IoT (versión preliminar)](quickstart-create-micro-agent-module-twin.md)
