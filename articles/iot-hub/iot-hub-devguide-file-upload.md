---
title: Información sobre la carga de archivos de Azure IoT Hub | Microsoft Docs
description: 'Guía del desarrollador: uso de la característica de carga de archivos de IoT Hub para administrar la carga de archivos desde un dispositivo a un contenedor de blobs de Azure Storage.'
author: robinsh
manager: philmea
ms.author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 11/07/2018
ms.custom:
- mqtt
- 'Role: Cloud Development'
- 'Role: IoT Device'
ms.openlocfilehash: bb0d39ea9e37f87a465ea5803e004a142c3a3fc6
ms.sourcegitcommit: 5da0bf89a039290326033f2aff26249bcac1fe17
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109715167"
---
# <a name="upload-files-with-iot-hub"></a>Carga de archivos con IoT Hub

Hay muchos escenarios en los que no se pueden asignar fácilmente los datos que los dispositivos envían en los mensajes del dispositivo a la nube con un tamaño relativamente reducido que acepte fácilmente IoT Hub. Por ejemplo:
* Archivos de imagen grandes
* Archivos de vídeo
* Datos de vibración muestreados con alta frecuencia
* Cierto tipo de datos procesados previamente

Cuando necesite cargar estos archivos desde un dispositivo, puede seguir utilizando la seguridad y confiabilidad de IoT Hub. En lugar de servir de intermediario de los mensajes, IoT Hub actúa como distribuidor a una cuenta de Azure Storage asociada. Un dispositivo solicita un token de almacenamiento de IoT Hub que es específico del archivo que el dispositivo quiere cargar. El dispositivo usa el URI de SAS para cargar el archivo en el almacenamiento y, cuando la carga ha finalizado, envía una notificación de finalización a IoT Hub. IoT Hub comprueba que se ha completado la carga de archivos.

[!INCLUDE [iot-hub-include-x509-ca-signed-file-upload-support-note](../../includes/iot-hub-include-x509-ca-signed-file-upload-support-note.md)]

### <a name="when-to-use"></a>Cuándo se usa

Cargas de archivos, para archivos multimedia y grandes lotes de telemetría cargados por dispositivos conectados de manera intermitente o comprimidos para ahorrar ancho de banda. Si duda entre el uso de propiedades notificadas, mensajes de dispositivo a nube o carga de archivos, consulte [Guía de comunicación de dispositivo a nube](iot-hub-devguide-d2c-guidance.md).

## <a name="associate-an-azure-storage-account-with-iot-hub"></a>Asociar una cuenta de Azure Storage con IoT Hub

Debe tener una cuenta de Azure Storage asociada a IoT Hub. 

Para aprender a crear una mediante el portal, consulte [Crear una cuenta de almacenamiento](../storage/common/storage-account-create.md). 

También puede crear una mediante programación por medio de las [API REST del proveedor de recursos de IoT Hub](/rest/api/iothub/iothubresource). 

Al asociar una cuenta de Azure Storage a un centro de IoT Hub, el centro genera un URI de SAS. Un dispositivo puede usar este URI de SAS para cargar de forma segura un archivo en un contenedor de blobs.

## <a name="create-a-container"></a>Crear un contenedor

 Para crear un contenedor de blobs a través del portal:

1. En el panel izquierdo de la cuenta de almacenamiento, en **Almacenamiento de datos**, seleccione **Contenedores**.
1. En la hoja Contenedor, seleccione **+ Contenedor**.
1. En el panel **Nuevo contenedor** que se abre, asigne un nombre al contenedor y seleccione **Crear**.

Después de crear un contenedor, siga las instrucciones que se detallan en [Configuración de cargas de archivos mediante Azure Portal](iot-hub-configure-file-upload.md). Asegúrese de que hay un contenedor de blobs asociado a su centro de IoT Hub y que las notificaciones de archivo están habilitadas.

También puede usar las [API REST del proveedor de recursos de IoT Hub](/rest/api/iothub/iothubresource) para crear un contenedor asociado al almacenamiento de IoT Hub.

## <a name="file-upload-using-an-sdk"></a>Carga de archivos mediante un SDK

Las siguientes guías paso a paso proporcionan tutoriales completos del proceso de carga de archivos en diversos lenguajes del SDK. Estas guías muestran cómo usar Azure Portal para asociar una cuenta de almacenamiento a un centro de IoT. También contienen fragmentos de código o hacen referencia a ejemplos que le guían a través del proceso de carga.

* [.NET](iot-hub-csharp-csharp-file-upload.md)
* [Java](iot-hub-java-java-file-upload.md)
* [Node.js](iot-hub-node-node-file-upload.md)
* [Python](iot-hub-python-python-file-upload.md)

> [!NOTE]
> Los [SDK de Azure IoT](iot-hub-devguide-sdks.md) administran automáticamente la recuperación del URI de firma de acceso compartido: cargan el archivo y notifican a IoT Hub que la carga se ha completado. Si un firewall bloquea el acceso al punto de conexión de Blob Storage, pero permite el acceso al punto de conexión de IoT Hub, se produce un error en el proceso de carga de archivos y se muestra el siguiente error para el SDK de dispositivo de C# de IoT:
>
> `---> System.Net.Http.HttpRequestException: A connection attempt failed because the connected party did not properly respond after a period of time, or established connection failed because connected host has failed to respond`
>
> Para que la característica de carga de archivos funcione, el acceso al punto de conexión de IoT Hub y al punto de conexión de Blob Storage debe estar disponible en el dispositivo.
> 


## <a name="initialize-a-file-upload-rest"></a>Inicialización de una carga de archivos (REST)

Puede usar las API REST en lugar de uno de los SDK para cargar un archivo. IoT Hub tiene un punto de conexión concreto para dispositivos para solicitar un URI de SAS para el almacenamiento para cargar un archivo. Para iniciar el proceso de carga de archivos, el dispositivo envía una solicitud POST a `{iot hub}.azure-devices.net/devices/{deviceId}/files` con el cuerpo JSON siguiente:

```json
{
    "blobName": "{name of the file for which a SAS URI will be generated}"
}
```

IoT Hub devuelve los datos siguientes, que usa el dispositivo para cargar el archivo:

```json
{
    "correlationId": "somecorrelationid",
    "hostName": "yourstorageaccount.blob.core.windows.net",
    "containerName": "testcontainer",
    "blobName": "test-device1/image.jpg",
    "sasToken": "1234asdfSAStoken"
}
```

### <a name="deprecated-initialize-a-file-upload-with-a-get"></a>En desuso: inicializar una carga de archivo con un comando GET

> [!NOTE]
> Esta sección describe la funcionalidad en desuso para la recepción de un URI de SAS de IoT Hub. Use el método POST que se ha descrito anteriormente.

IoT Hub tiene dos puntos de conexión de REST que permiten la carga de archivos: uno para obtener el URI de SAS para el almacenamiento y el otro para notificar a IoT Hub que la carga se ha completado. El dispositivo inicia el proceso de carga de archivos mediante el envío de una operación GET al centro de IoT en `{iot hub}.azure-devices.net/devices/{deviceId}/files/{filename}`. IoT Hub devuelve:

* Un URI de SAS específico del archivo que se va a cargar.

* Un identificador de correlación que se usará cuando se haya completado la carga.

## <a name="notify-iot-hub-of-a-completed-file-upload-rest"></a>Notificación a IoT Hub de una carga de archivos completada (REST)

El dispositivo carga el archivo en el almacenamiento mediante los SDK de Azure Storage. Cuando la carga se haya completado, el dispositivo envía una solicitud POST a `{iot hub}.azure-devices.net/devices/{deviceId}/files/notifications` con el siguiente cuerpo JSON:

```json
{
    "correlationId": "{correlation ID received from the initial request}",
    "isSuccess": bool,
    "statusCode": XXX,
    "statusDescription": "Description of status"
}
```

El valor de `isSuccess` es un valor booleano que indica si se cargó correctamente el archivo. El código de estado para `statusCode` es el estado de la carga del archivo al almacenamiento y `statusDescription` corresponde a `statusCode`.

## <a name="reference-topics"></a>Temas de referencia:

Los siguientes temas de referencia proporcionan más información sobre la carga de archivos desde un dispositivo.

### <a name="file-upload-notifications"></a>Notificaciones de carga de archivos

Opcionalmente, cuando un dispositivo notifica a IoT Hub que la carga está completa, IoT Hub genera un mensaje de notificación. Este mensaje contiene el nombre y ubicación de almacenamiento del archivo.

Como se ha explicado en [Puntos de conexión](iot-hub-devguide-endpoints.md), IoT Hub entrega notificaciones de carga de archivos a través de un punto de conexión accesible desde el servicio (**/messages/servicebound/fileuploadnotifications**) en forma de mensajes. La semántica de recepción de las notificaciones de carga de archivos es la misma que para los mensajes de nube a dispositivo y tiene el mismo [ciclo de vida del mensaje](iot-hub-devguide-messages-c2d.md#the-cloud-to-device-message-life-cycle). Cada mensaje recuperado del punto de conexión de notificación de carga de archivos es un registro JSON con las siguientes propiedades:

| Propiedad | Descripción |
| --- | --- |
| EnqueuedTimeUtc |Marca de tiempo que indica cuándo se creó la notificación. |
| deviceId |**DeviceId** del dispositivo que ha cargado el archivo. |
| BlobUri |URI del archivo cargado. |
| BlobName |Nombre del archivo cargado. |
| LastUpdatedTime |Marca de tiempo que indica cuándo se actualizó por última vez el archivo. |
| BlobSizeInBytes |Tamaño del archivo cargado. |

**Ejemplo**. Este ejemplo muestra el cuerpo de ejemplo de un mensaje de notificación de carga de archivos.

```json
{
    "deviceId":"mydevice",
    "blobUri":"https://{storage account}.blob.core.windows.net/{container name}/mydevice/myfile.jpg",
    "blobName":"mydevice/myfile.jpg",
    "lastUpdatedTime":"2016-06-01T21:22:41+00:00",
    "blobSizeInBytes":1234,
    "enqueuedTimeUtc":"2016-06-01T21:22:43.7996883Z"
}
```

### <a name="file-upload-notification-configuration-options"></a>Opciones de configuración de notificaciones de carga de archivos

Cada centro de IoT tiene las siguientes opciones de configuración para las notificaciones de carga de archivos:

| Propiedad | Descripción | Intervalo y valor predeterminado |
| --- | --- | --- |
| **enableFileUploadNotifications** |Controla si las notificaciones de carga de archivos se escriben en el punto de conexión de notificaciones de archivo. |Bool. Valor predeterminado: True. |
| **fileNotifications.ttlAsIso8601** |TTL predeterminado para las notificaciones de carga de archivos. |Intervalo ISO_8601 hasta 48H (1 minuto como mínimo). Valor predeterminado: 1 hora. |
| **fileNotifications.lockDuration** |Duración del bloqueo de la cola de notificaciones de carga de archivos. |De 5 a 300 segundos (5 segundos como mínimo). Valor predeterminado: 60 segundos. |
| **fileNotifications.maxDeliveryCount** |Número máximo de entregas en la cola de notificaciones de carga de archivos. |De 1 a 100. Valor predeterminado: 100. |

Puede establecer estas propiedades en la instancia de IoT Hub mediante Azure Portal, la CLI de Azure o PowerShell. Para más información sobre cómo hacerlo, consulte los temas que se encuentran en [Configuración de la carga de archivos](iot-hub-configure-file-upload.md).

## <a name="additional-reference-material"></a>Material de referencia adicional

Otros temas de referencia en la guía del desarrollador de IoT Hub son los siguientes:

* En [Puntos de conexión de IoT Hub](iot-hub-devguide-endpoints.md) se describen los diferentes puntos de conexión del centro de IoT para las operaciones en tiempo de ejecución y de administración.

* En [Cuotas y limitación](iot-hub-devguide-quotas-throttling.md) se describen las cuotas y el comportamiento de limitación que se aplican al servicio IoT Hub.

* En [SDK de dispositivo y servicio IoT de Azure](iot-hub-devguide-sdks.md) se muestran los diversos SDK de lenguaje que puede usar para desarrollar aplicaciones para dispositivo y de servicio que interactúen con IoT Hub.

* En [Lenguaje de consulta de IoT Hub](iot-hub-devguide-query-language.md) se describe el lenguaje de consulta que se puede usar para recuperar información de IoT Hub sobre los trabajos y dispositivos gemelos.

* En [Compatibilidad con MQTT de IoT Hub](iot-hub-mqtt-support.md) se proporciona más información sobre la compatibilidad de IoT Hub con el protocolo MQTT.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha aprendido a cargar los archivos desde dispositivos con IoT Hub, pueden interesarle los siguientes temas de la guía del desarrollador de IoT Hub:

* [Administrar identidades del dispositivo en IoT Hub](iot-hub-devguide-identity-registry.md)

* [Control del acceso a IoT Hub](iot-hub-devguide-security.md)

* [Uso de dispositivos gemelos para sincronizar el estado y las configuraciones](iot-hub-devguide-device-twins.md)

* [Invocación de un método directo en un dispositivo](iot-hub-devguide-direct-methods.md)

* [Programación de trabajos en varios dispositivos](iot-hub-devguide-jobs.md)

Para probar algunos de los conceptos descritos en este artículo, vea el siguiente tutorial de IoT Hub:

* [Carga de archivos desde dispositivos a la nube con IoT Hub](iot-hub-csharp-csharp-file-upload.md)
