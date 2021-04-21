---
title: 'Inicio rápido: Configuración de reglas y acciones en Azure IoT Central'
description: En esta guía de inicio rápido se muestra cómo puede, como generador, configurar las reglas y las acciones basadas en la telemetría en la aplicación de Azure IoT Central.
author: dominicbetts
ms.author: dobett
ms.date: 11/16/2020
ms.topic: quickstart
ms.service: iot-central
services: iot-central
ms.custom: mvc
ms.openlocfilehash: f65614de97e8ff6eed732e624ae30c3f2b70bd60
ms.sourcegitcommit: 272351402a140422205ff50b59f80d3c6758f6f6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/17/2021
ms.locfileid: "107589013"
---
# <a name="quickstart-configure-rules-and-actions-for-your-device-in-azure-iot-central"></a>Inicio rápido: Configuración de reglas y acciones para el dispositivo en Azure IoT Central

En este inicio rápido se crea una regla que envía un correo cuando la humedad que notifique el sensor de algún dispositivo supere el 55 %.

## <a name="prerequisites"></a>Prerrequisitos

Antes de comenzar debe completar los dos inicios rápidos anteriores [Creación de una aplicación de Azure IoT Central](./quick-deploy-iot-central.md) e [Incorporación de un dispositivo simulado a una aplicación de IoT Central](./quick-create-simulated-device.md) para crear la plantilla de dispositivo **Sensor Controller** con la que trabajar.

## <a name="create-a-telemetry-based-rule"></a>Crear una regla basada en la telemetría

1. Para agregar una nueva regla basada en telemetría a la aplicación, en el panel izquierdo, seleccione **Reglas**.

1. Para crear una regla, seleccione **+ New** (+ Nuevo).

1. Especifique **Environmental humidity** (Humedad ambiental) como nombre de la regla.

1. En la sección **Target devices** (Dispositivos de destino), seleccione **Sensor Controller** como plantilla de dispositivo. Esta opción filtra los dispositivos a los que se aplica la regla por tipo de plantilla de dispositivo. Para agregar más criterios de filtro, seleccione **+ Filtro**.

1. En la sección **Condiciones**, se define lo que desencadena la regla. Use la siguiente información para definir una condición basada en los datos de telemetría de temperatura:

    | Campo        | Value            |
    | ------------ | ---------------- |
    | Medición  | SensorHumid      |
    | Operator     | es mayor que  |
    | Value        | 55               |

    Para agregar más condiciones, seleccione **+ Condition** (+ Condición).

    :::image type="content" source="media/quick-configure-rules/condition.png" alt-text="Captura de pantalla que muestra la condición de la regla.":::

1. Para agregar una acción de correo para que se ejecute cuando se desencadene la regla, seleccione **+ Correo electrónico**.

1. Use la información de la tabla siguiente para definir la acción y, después, seleccione **Listo**:

    | Configuración   | Value                                             |
    | --------- | ------------------------------------------------- |
    | Nombre para mostrar | Adición de una acción de correo electrónico                          |
    | A        | La dirección de correo electrónico propia                                |
    | Notas     | La humedad ambiental superó el umbral. |

    > [!NOTE]
    > Para recibir una notificación por correo electrónico, la dirección debe ser un [identificador de usuario de la aplicación](howto-administer.md) y ese usuario debe haber iniciado sesión en la aplicación al menos una vez.

    :::image type="content" source="media/quick-configure-rules/action.png" alt-text="Captura de pantalla que muestra la acción de correo electrónico agregada a la regla.":::

1. Seleccione **Guardar**. La regla se muestra en la página **Reglas**.

## <a name="test-the-rule"></a>Prueba de la regla

Poco después de guardar la regla, esta se activa. Cuando se cumplan las condiciones definidas en la regla, la aplicación enviará un correo electrónico a la dirección que especificó en la acción.

> [!NOTE]
> Una vez terminada la prueba, desactive la regla para dejar de recibir alertas en la bandeja de entrada.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a hacer lo siguiente:

* Crear una regla basada en la telemetría
* Agregar una acción

Para más información sobre la supervisión de dispositivos conectados a la aplicación, continúe con la guía de inicio rápido:

> [!div class="nextstepaction"]
> [Uso de Azure IoT Central para supervisar los dispositivos](quick-monitor-devices.md).
