---
title: archivo de inclusión
description: archivo de inclusión
author: timlt
ms.service: iot-develop
ms.topic: include
ms.date: 05/17/2021
ms.author: timlt
ms.custom: include file
ms.openlocfilehash: 3620e8021cfe5adf6226f007ea240921951ea6fe
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110061306"
---
## <a name="create-the-cloud-components"></a>Creación de los componentes de nube

### <a name="create-the-iot-central-application"></a>Creación de la aplicación de IoT Central

Existen varias formas de conectar los dispositivos a Azure IoT. En esta sección, aprenderá a conectar un dispositivo con Azure IoT Central. IoT Central es una plataforma de aplicaciones IoT que reduce la complejidad y el costo de crear y administrar soluciones IoT.

Para crear una aplicación:
1. En el [portal de Azure IoT Central](https://apps.azureiotcentral.com/), seleccione **Aplicaciones** en el menú de navegación lateral.
1. Seleccione **Nueva aplicación**.
1. Seleccione **Aplicaciones personalizadas**.
1. Agregue el nombre de la aplicación y una dirección URL.
1. Elija el plan de precios **Gratuito** para activar una versión de prueba de 7 días.

    :::image type="content" source="media/iot-develop-embedded-create-central-app-with-device/iot-central-create-custom.png" alt-text="Creación de una aplicación personalizada en Azure IoT Central":::

1. Seleccione **Crear**.

    Después de que IoT Central aprovisione la aplicación, le redirigirá automáticamente al panel de la nueva aplicación.

    > [!NOTE]
    > Si ya tiene una aplicación de IoT Central, puede usarla para realizar los pasos de este artículo en lugar de crear otra.

### <a name="create-a-new-device"></a>Creación de un dispositivo

En esta sección, usará el panel de la aplicación de IoT Central para crear un dispositivo. Usará la información de conexión del dispositivo recién creado para conectar de forma segura el dispositivo físico en una sección posterior.

Para crear un dispositivo:
1. En el panel de la aplicación, seleccione **Dispositivos** en el menú de navegación lateral.
1. Seleccione **+ Nuevo** para abrir la ventana **Crear un nuevo dispositivo**.
1. Deje la plantilla Dispositivo como **Sin asignar**.
1. Rellene el nombre y el identificador del dispositivo.

    :::image type="content" source="media/iot-develop-embedded-create-central-app-with-device/iot-central-create-device.png" alt-text="Creación de un dispositivo en Azure IoT Central":::

1. Seleccione el botón **Crear**.
1. El dispositivo recién creado aparece en la lista **Todos los dispositivos**.  Seleccione el nombre del dispositivo para ver los detalles.
1. Seleccione **Conectar** en la barra de menús superior derecha para mostrar la información de conexión que se va a usar para configurar el dispositivo en la próxima sección.

    :::image type="content" source="media/iot-develop-embedded-create-central-app-with-device/iot-central-device-connection-info.png" alt-text="Vista de detalles de conexión del dispositivo":::

1. Observe los valores de conexión de los siguientes parámetros de cadena de conexión que aparecen en el cuadro de diálogo **Conectar**. Agregará estos valores a un archivo de configuración en el próximo paso:

    > * `ID scope`
    > * `Device ID`
    > * `Primary key`