---
title: Carga de archivos desde dispositivos a Azure Storage | Microsoft Docs
description: Cómo configurar cargas de archivos desde los dispositivos a la nube. Después de configurar las cargas de archivos, implemente cargas de archivos a sus dispositivos.
services: iot-central
author: dominicbetts
ms.author: dobett
ms.date: 12/23/2020
ms.topic: how-to
ms.service: iot-central
ms.openlocfilehash: 1cca508e213d132f96a131f1b0029c13e1b67c47
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108760794"
---
# <a name="upload-files-from-your-devices-to-the-cloud"></a>Cargar archivos desde dispositivos a la nube

IoT Central le permite cargar medios y otros archivos desde dispositivos conectados al almacenamiento en la nube. Configure la funcionalidad de carga de archivos en la aplicación IoT Central y a continuación implemente cargas de archivos en el código del dispositivo.

## <a name="prerequisites"></a>Prerrequisitos

Para configurar la carga de archivos debe ser administrador en la aplicación de IoT Central.

Necesita una cuenta en Azure Storage y un contenedor para almacenar los archivos cargados. Si no dispone de una cuenta de almacenamiento y un contenedor para usar, [cree una en Azure Portal](https://ms.portal.azure.com/#create/Microsoft.StorageAccount-ARM).

## <a name="configure-device-file-uploads"></a>Configurar cargas de archivos del dispositivo

Para configurar cargas de archivos del dispositivo:

1. Vaya a la sección **Administración** en su aplicación.

1. Seleccione **Carga de archivos del dispositivo**.

1. Vaya a la cuenta de almacenamiento y seleccione el contenedor. Si la cuenta de almacenamiento está en una suscripción de Azure diferente de la aplicación, introduzca una cadena de conexión de la cuenta de almacenamiento.

1. Si es necesario, ajuste el tiempo de espera de carga que establece el tiempo durante el que una solicitud de carga es válida. Los valores válidos van de 1 a 24 horas.

1. Seleccione **Guardar**. Cuando el estado muestra **Configurado**, está listo para cargar archivos desde dispositivos.

:::image type="content" source="media/howto-configure-file-uploads/file-upload-configuration.png" alt-text="Configurar la carga de archivos en la aplicación":::

## <a name="disable-device-file-uploads"></a>Configurar cargas de archivos del dispositivo

Si desea deshabilitar las cargas de archivos de dispositivo en la aplicación IoT Central:

1. Vaya a la sección **Administración** en su aplicación.

1. Seleccione **Carga de archivos del dispositivo**.

1. Seleccione **Eliminar**.

## <a name="upload-a-file-from-a-device"></a>Carga de un archivo desde un dispositivo

IoT Central usa la capacidad de carga de archivos de IoT Hub para permitir que los dispositivos carguen archivos. Para ver el código de ejemplo que muestra cómo cargar archivos desde un dispositivo, consulte el[ejemplo de la carga de archivos de IoT Central](/samples/iot-for-all/iotc-file-upload-device/iotc-file-upload-device/).

## <a name="next-steps"></a>Pasos siguientes

Ahora que sabe cómo configurar e implementar cargas de archivos de dispositivo en IoT Central, el siguiente paso que se sugiere es aprender más cargas de archivos de dispositivo:

- [Carga de archivos de un dispositivo a la nube con IoT Hub (.NET)](../../iot-hub/iot-hub-csharp-csharp-file-upload.md)
- [Carga de archivos de un dispositivo a la nube con IoT Hub (Java)](../../iot-hub/iot-hub-java-java-file-upload.md)
- [Carga de archivos de un dispositivo a la nube con IoT Hub (Node.js)](../../iot-hub/iot-hub-node-node-file-upload.md)
- [Carga de archivos de un dispositivo a la nube con IoT Hub (Python)](../../iot-hub/iot-hub-python-python-file-upload.md)