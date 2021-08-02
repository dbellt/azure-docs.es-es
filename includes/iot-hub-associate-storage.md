---
title: archivo de inclusión
description: archivo de inclusión
services: iot-edge
author: kgremban
ms.service: iot-edge
ms.topic: include
ms.date: 01/22/2019
ms.author: kgremban
ms.custom: include file
ms.openlocfilehash: ecd7e070c353d706865b245427758c11a7bbfa5d
ms.sourcegitcommit: 34feb2a5bdba1351d9fc375c46e62aa40bbd5a1f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111896875"
---
## <a name="associate-an-azure-storage-account-to-iot-hub"></a>Asociación de una cuenta de Azure Storage a IoT Hub

Debe tener una cuenta de Azure Storage asociada a IoT Hub. Para aprender a crear una, consulte [Creación de una cuenta de Storage](../articles/storage/common/storage-account-create.md). Al asociar una cuenta de Azure Storage a un centro de IoT Hub, el centro genera un URI de SAS. Un dispositivo puede usar este URI de SAS para cargar de forma segura un archivo en un contenedor de blobs. El servicio IoT Hub y los SDK de dispositivo coordinan el proceso que genera el URI de SAS y permite que esté disponible para un dispositivo para cargar un archivo.

## <a name="create-a-container"></a>Creación de un contenedor

Siga estos pasos para crear un contenedor de blobs para la cuenta de almacenamiento:

1. En el panel izquierdo de la cuenta de almacenamiento, en **Almacenamiento de datos**, seleccione **Contenedores**.
1. En la hoja Contenedor, seleccione **+ Contenedor**.
1. En el panel **Nuevo contenedor** que se abre, asigne un nombre al contenedor y seleccione **Crear**.

Después de crear un contenedor, siga las instrucciones que se detallan en [Configuración de cargas de archivos mediante Azure Portal](../articles/iot-hub/iot-hub-configure-file-upload.md). Asegúrese de que hay un contenedor de blobs asociado a su centro de IoT Hub y que las notificaciones de archivo están habilitadas.

![Habilitación de las notificaciones de archivo en el Portal](./media/iot-hub-associate-storage/file-notifications-vs2019.png)
