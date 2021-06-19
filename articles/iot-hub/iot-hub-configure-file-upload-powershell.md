---
title: Uso de Azure PowerShell para configurar la carga de archivos | Microsoft Docs
description: Cómo usar los cmdlets de Azure PowerShell para configurar su centro de IoT para habilitar cargas de archivos desde dispositivos conectados. Incluye información sobre cómo configurar la cuenta de Azure Storage.
author: robinsh
manager: philmea
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 08/08/2017
ms.author: robinsh
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: dc0efdd9eb4951ef65c3eb771e17731213dfbcc2
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110702242"
---
# <a name="configure-iot-hub-file-uploads-using-powershell"></a>Configuración de cargas de archivos de IoT Hub mediante PowerShell

[!INCLUDE [iot-hub-file-upload-selector](../../includes/iot-hub-file-upload-selector.md)]

Para usar la [funcionalidad de carga de archivos en IoT Hub](iot-hub-devguide-file-upload.md), primero debe asociar una cuenta de Azure Storage con IoT Hub. Puede usar una cuenta de almacenamiento existente o crear una nueva.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Para completar este tutorial, necesitará lo siguiente:

* Una cuenta de Azure activa. En caso de no tener ninguna, puede crear una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/) en tan solo unos minutos.

* [Cmdlets de Azure PowerShell](/powershell/azure/install-Az-ps).

* Un centro de Azure IoT. Si no dispone de un centro de IoT, puede usar el [cmdlet New-AzIoTHub](/powershell/module/az.iothub/new-aziothub) para crear uno o usar el portal para [crear un centro de IoT](iot-hub-create-through-portal.md).

* Una cuenta de almacenamiento de Azure. Si no tiene una cuenta de almacenamiento de Azure, puede usar los [cmdlets de PowerShell de Azure Storage](/powershell/module/az.storage/) para crear una o usar el portal para [crear una cuenta de almacenamiento](../storage/common/storage-account-create.md).

## <a name="sign-in-and-set-your-azure-account"></a>Inicio de sesión y configuración de la cuenta de Azure

Inicie sesión en la cuenta de Azure y seleccione su suscripción.

1. En el símbolo del sistema de PowerShell, ejecute el cmdlet **Connect-AzAccount**:

    ```powershell
    Connect-AzAccount
    ```

2. Si tiene varias suscripciones de Azure, el inicio de sesión en Azure le concede acceso a todas las suscripciones de Azure asociadas a sus credenciales. Use el siguiente comando para mostrar las suscripciones de Azure que están disponibles para su uso:

    ```powershell
    Get-AzSubscription
    ```

    Use el siguiente comando para seleccionar la suscripción que desea usar para ejecutar los comandos para administrar su centro de IoT. Puede usar el nombre de la suscripción o el identificador de la salida del comando anterior:

    ```powershell
    Select-AzSubscription `
        -SubscriptionName "{your subscription name}"
    ```

## <a name="retrieve-your-storage-account-details"></a>Recuperación de los detalles de la cuenta de almacenamiento

En los siguientes pasos se supone que ha creado la cuenta de almacenamiento mediante el modelo de implementación de **Resource Manager**, y no el modelo **clásico**.

Para configurar cargas de archivos desde sus dispositivos, necesitará la cadena de conexión de una cuenta de almacenamiento de Azure. Esta cuenta debe encontrarse en la misma suscripción que IoT Hub. También necesitará el nombre de un contenedor de blobs de la cuenta de almacenamiento. Use el siguiente comando para recuperar las claves de la cuenta de almacenamiento:

```powershell
Get-AzStorageAccountKey `
  -Name {your storage account name} `
  -ResourceGroupName {your storage account resource group}
```

Anote el valor de la clave de almacenamiento **key1**. La necesitará en los pasos siguientes.

Puede usar un contenedor de blobs existente para sus cargas de archivos o crear uno nuevo:

* Para mostrar los contenedores de blobs existentes en su cuenta de almacenamiento, use los siguientes comandos:

    ```powershell
    $ctx = New-AzStorageContext `
        -StorageAccountName {your storage account name} `
        -StorageAccountKey {your storage account key}
    Get-AzStorageContainer -Context $ctx
    ```

* Para crear un contenedor de blobs en su cuenta de almacenamiento, use los siguientes comandos:

    ```powershell
    $ctx = New-AzStorageContext `
        -StorageAccountName {your storage account name} `
        -StorageAccountKey {your storage account key}
    New-AzStorageContainer `
        -Name {your new container name} `
        -Permission Off `
        -Context $ctx
    ```

## <a name="configure-your-iot-hub"></a>Configuración del centro de IoT

Ahora puede configurar el centro de IoT para [cargar archivos en el centro de IoT](iot-hub-devguide-file-upload.md) con los datos de su cuenta de almacenamiento.

La configuración requiere los siguientes valores:

* **Contenedor de almacenamiento:** : un contenedor de blobs en una cuenta de Azure Storage en la suscripción actual para asociar con IoT Hub. En la sección anterior, recuperó la información necesaria de la cuenta de almacenamiento. IoT Hub genera automáticamente identificadores URI de SAS con permisos de escritura en este contenedor de blobs para los dispositivos que se utilizarán cuando se carguen archivos.

* **Receive notifications for uploaded files** (Recibir notificaciones para archivos cargados): habilite o deshabilite las notificaciones de carga de archivos.

* **SAS TTL**(TTL SAS): este valor es el periodo de vida de los URI de SAS que Azure IoT Hub devuelve al dispositivo. De forma predeterminada, está establecido en una hora.

* **File notification settings default TTL**(TTL predeterminado de configuración de notificación de archivos): el periodo de vida de una notificación de carga de archivos antes de que caduque. De forma predeterminada, está establecido en un día.

* **File notification maximum delivery count**(Número máximo de entregas de notificaciones de archivo): el número de veces que Azure IoT Hub tratará de entregar una notificación de carga de archivos. De forma predeterminada, está establecido en 10.

Use el siguiente cmdlet de PowerShell para configurar la carga de archivos en su centro de IoT:

```powershell
Set-AzIotHub `
    -ResourceGroupName "{your iot hub resource group}" `
    -Name "{your iot hub name}" `
    -FileUploadNotificationTtl "01:00:00" `
    -FileUploadSasUriTtl "01:00:00" `
    -EnableFileUploadNotifications $true `
    -FileUploadStorageConnectionString "DefaultEndpointsProtocol=https;AccountName={your storage account name};AccountKey={your storage account key};EndpointSuffix=core.windows.net" `
    -FileUploadContainerName "{your blob container name}" `
    -FileUploadNotificationMaxDeliveryCount 10
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
