---
title: Configuración de la carga de archivos en IoT Hub con la CLI de Azure | Microsoft Docs
description: Configuración de las cargas de archivos en Azure IoT Hub con la CLI de Azure multiplataforma.
author: robinsh
manager: philmea
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 08/08/2017
ms.author: robinsh
ms.openlocfilehash: ea6ec30ad5f3b1cdbc906cc94cb211295b84e802
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107761734"
---
# <a name="configure-iot-hub-file-uploads-using-azure-cli"></a>Configuración de cargas de archivos de IoT Hub mediante la CLI de Azure

[!INCLUDE [iot-hub-file-upload-selector](../../includes/iot-hub-file-upload-selector.md)]

Para [cargar archivos desde un dispositivo](iot-hub-devguide-file-upload.md), primero debe asociar una cuenta de Azure Storage con la instancia de IoT Hub. Puede usar una cuenta de almacenamiento existente o crear una nueva.

Para completar este tutorial, necesitará lo siguiente:

* Una cuenta de Azure activa. En caso de no tener ninguna, puede crear una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/) en tan solo unos minutos.

* [Azure CLI](/cli/azure/install-azure-cli).

* Un centro de Azure IoT. Si no tiene instancias de IoT Hub, puede usar el [`az iot hub create` comando](/cli/azure/iot/hub#az_iot_hub_create) para crear una o [crear una instancia de IoT Hub mediante el portal](iot-hub-create-through-portal.md).

* Una cuenta de Azure Storage. Si no tiene ninguna cuenta de Azure Storage, puede usar la CLI de Azure para crear una. Para obtener más información, consulte [Creación de una cuenta de almacenamiento](../storage/common/storage-account-create.md).

## <a name="sign-in-and-set-your-azure-account"></a>Inicio de sesión y configuración de la cuenta de Azure

Inicie sesión en la cuenta de Azure y seleccione su suscripción.

1. En el símbolo del sistema, ejecute el [comando de inicio de sesión](/cli/azure/get-started-with-azure-cli):

    ```azurecli
    az login
    ```

    Siga las instrucciones para realizar la autenticación mediante el código e inicie sesión en la cuenta de Azure a través de un explorador web.

2. Si tiene varias suscripciones de Azure, iniciar sesión en Azure le concede acceso a todas las cuentas de Azure asociadas con las credenciales. Use el siguiente [comando para mostrar las cuentas de Azure](/cli/azure/account) que tiene disponibles para su uso:

    ```azurecli
    az account list
    ```

    Use el siguiente comando para seleccionar la suscripción que desee usar para ejecutar los comandos que crearán la instancia de IoT Hub. Puede usar el nombre de la suscripción o el identificador de la salida del comando anterior:

    ```azurecli
    az account set --subscription {your subscription name or id}
    ```

## <a name="retrieve-your-storage-account-details"></a>Recuperación de los detalles de la cuenta de almacenamiento

En los siguientes pasos se supone que ha creado la cuenta de almacenamiento mediante el modelo de implementación de **Resource Manager**, y no el modelo **clásico**.

Para configurar cargas de archivos desde sus dispositivos, necesitará la cadena de conexión de una cuenta de almacenamiento de Azure. Esta cuenta debe encontrarse en la misma suscripción que IoT Hub. También necesitará el nombre de un contenedor de blobs de la cuenta de almacenamiento. Use el siguiente comando para recuperar las claves de la cuenta de almacenamiento:

```azurecli
az storage account show-connection-string --name {your storage account name} \
  --resource-group {your storage account resource group}
```

Anote el valor de **connectionString**. La necesitará en los pasos siguientes.

Puede usar un contenedor de blobs existente para sus cargas de archivos o crear uno nuevo:

* Para mostrar los contenedores de blobs existentes en su cuenta de almacenamiento, use el siguiente comando:

    ```azurecli
    az storage container list --connection-string "{your storage account connection string}"
    ```

* Para crear un contenedor de blobs en su cuenta de almacenamiento, use el siguiente comando:

    ```azurecli
    az storage container create --name {container name} \
      --connection-string "{your storage account connection string}"
    ```

## <a name="file-upload"></a>Carga de archivos

Ahora puede configurar la instancia de IoT Hub para permitir la [carga de archivos en IoT Hub](iot-hub-devguide-file-upload.md) con los datos de la cuenta de almacenamiento.

La configuración requiere los siguientes valores:

* **Contenedor de almacenamiento:** : un contenedor de blobs en una cuenta de Azure Storage en la suscripción actual para asociar con IoT Hub. En la sección anterior, recuperó la información necesaria de la cuenta de almacenamiento. IoT Hub genera automáticamente identificadores URI de SAS con permisos de escritura en este contenedor de blobs para los dispositivos que se utilizarán cuando se carguen archivos.

* **Receive notifications for uploaded files** (Recibir notificaciones para archivos cargados): habilite o deshabilite las notificaciones de carga de archivos.

* **SAS TTL**(TTL SAS): este valor es el periodo de vida de los URI de SAS que Azure IoT Hub devuelve al dispositivo. De forma predeterminada, está establecido en una hora.

* **File notification settings default TTL**(TTL predeterminado de configuración de notificación de archivos): el periodo de vida de una notificación de carga de archivos antes de que caduque. De forma predeterminada, está establecido en un día.

* **File notification maximum delivery count**(Número máximo de entregas de notificaciones de archivo): el número de veces que Azure IoT Hub tratará de entregar una notificación de carga de archivos. De forma predeterminada, está establecido en 10.

Use los siguientes comandos del CLI de Azure para configurar la carga de archivos en IoT Hub:

<!--Robinsh this is out of date, add cloud powershell -->

En un shell de Bash, use:

```azurecli
az iot hub update --name {your iot hub name} \
  --set properties.storageEndpoints.'$default'.connectionString="{your storage account connection string}"

az iot hub update --name {your iot hub name} \
  --set properties.storageEndpoints.'$default'.containerName="{your storage container name}"

az iot hub update --name {your iot hub name} \
  --set properties.storageEndpoints.'$default'.sasTtlAsIso8601=PT1H0M0S

az iot hub update --name {your iot hub name} \
  --set properties.enableFileUploadNotifications=true

az iot hub update --name {your iot hub name} \
  --set properties.messagingEndpoints.fileNotifications.maxDeliveryCount=10

az iot hub update --name {your iot hub name} \
  --set properties.messagingEndpoints.fileNotifications.ttlAsIso8601=PT1H0M0S
```

Puede revisar la configuración de carga de archivos en IoT Hub con el siguiente comando:

```azurecli
az iot hub show --name {your iot hub name}
```

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre las funcionalidades de carga de archivos de IoT Hub, consulte el artículo sobre la [carga de archivos desde un dispositivo](iot-hub-devguide-file-upload.md).

Siga estos vínculos para más información sobre la administración de Azure IoT Hub:

* [Administración de identidades de dispositivos de Centro de IoT de forma masiva](iot-hub-bulk-identity-mgmt.md)
* [Supervisión del centro de IoT](monitor-iot-hub.md)

Para explorar aún más las funcionalidades de IoT Hub, consulte:

* [Guía para desarrolladores de IoT Hub](iot-hub-devguide.md)
* [Implementación de IA en dispositivos perimetrales con Azure IoT Edge](../iot-edge/quickstart-linux.md)
* [Protección total de la solución de IoT](../iot-fundamentals/iot-security-ground-up.md)