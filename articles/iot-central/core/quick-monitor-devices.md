---
title: 'Inicio rápido: Supervisión de dispositivos en Azure IoT Central'
description: 'Inicio rápido: Como operador, aprenda a usar una aplicación de Azure IoT Central para supervisar los dispositivos.'
author: dominicbetts
ms.author: dobett
ms.date: 11/16/2020
ms.topic: quickstart
ms.service: iot-central
services: iot-central
ms.custom: mvc
ms.openlocfilehash: 4c3aaeb5b3c01c9c940d49fd7022de653de0e48b
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108206788"
---
# <a name="quickstart-use-azure-iot-central-to-monitor-your-devices"></a>Inicio rápido: Uso de Azure IoT Central para supervisar los dispositivos

*Este artículo se aplica a los administradores, operadores y compiladores.*

En este inicio rápido se muestra cómo usted, como operador, puede usar la aplicación de Azure IoT Central para supervisar los dispositivos y cambiar la configuración.

## <a name="prerequisites"></a>Prerrequisitos

Antes de comenzar, debe completar los tres inicios rápidos anteriores: [Crear una aplicación de Azure IoT Central](./quick-deploy-iot-central.md), [Agregar un dispositivo simulado a la aplicación en IoT Central](./quick-create-simulated-device.md) y [Configurar reglas y acciones para el dispositivo](quick-configure-rules.md).

## <a name="receive-a-notification"></a>Recibir una notificación

Azure IoT Central envía notificaciones acerca de los dispositivos como mensajes de correo electrónico. Como creador, ha agregado una regla para enviar una notificación al operador cuando la humedad del sensor de cualquier dispositivo conectado supere un umbral. Como operador, comprueba los mensajes de correo electrónico en busca de notificaciones.

Abra el mensaje de correo electrónico que recibió al final del inicio rápido [Configurar reglas y acciones para el dispositivo](quick-configure-rules.md). En el correo electrónico, seleccione el vínculo al dispositivo:

:::image type="content" source="media/quick-monitor-devices/email.png" alt-text="Captura de pantalla que muestra un correo electrónico de notificación.":::

Se abre en el explorador la vista **Información general** del dispositivo simulado que creó en los inicios rápidos anteriores:

:::image type="content" source="media/quick-monitor-devices/dashboard.png" alt-text="Captura de pantalla que muestra la información general del dispositivo que desencadenó la notificación.":::

## <a name="investigate-an-issue"></a>Investigar un problema

Los operadores pueden ver información acerca del dispositivo en las vistas **Información general**, **Acerca de** y **Comandos**. El generador ha creado una vista **Administrar dispositivo** en la que se puede editar la información del dispositivo y establecer sus propiedades.

El gráfico del panel muestra un trazado de la humedad del dispositivo. El usuario decide que la humedad del dispositivo es demasiada.

## <a name="remediate-an-issue"></a>Corregir un problema

Para realizar cualquier cambio en el dispositivo, use la página **Administrar dispositivo**.

Cambie **Target temperature** (Temperatura objetivo) a 80 para que el dispositivo esté caliente y reduzca la humedad. Elija **Guardar** para actualizar el dispositivo. Cuando el dispositivo confirma el cambio en la configuración, el estado del valor cambia a **sincronizado**:

:::image type="content" source="media/quick-monitor-devices/change-settings.png" alt-text="Captura de pantalla que muestra la configuración de temperatura objetivo actualizada para el dispositivo.":::

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [iot-central-clean-up-resources](../../../includes/iot-central-clean-up-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a hacer lo siguiente:

* Recibir una notificación
* Investigar un problema
* Corregir un problema

Ahora que sabe cómo supervisar un dispositivo, el siguiente paso que se recomienda es:

> [!div class="nextstepaction"]
> [Cree y administre una plantilla de dispositivo](howto-set-up-template.md).
