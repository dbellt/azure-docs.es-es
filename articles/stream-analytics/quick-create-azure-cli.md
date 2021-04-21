---
title: 'Inicio rápido: Creación de un trabajo de Azure Stream Analytics mediante la CLI de Azure'
description: En este inicio rápido se muestra cómo usar la CLI de Azure para crear un trabajo Azure Stream Analytics.
services: stream-analytics
ms.service: stream-analytics
author: sidramadoss
ms.author: sidram
ms.reviewer: jasonh
ms.workload: big-data
ms.topic: quickstart
ms.custom: mvc, devx-track-azurecli
ms.date: 07/01/2020
ms.openlocfilehash: 58dccf56cd493782a422b0ddf0386e31d4d87daf
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107765996"
---
# <a name="quickstart-create-an-azure-stream-analytics-job-using-the-azure-cli"></a>Inicio rápido: Creación de un trabajo de Azure Stream Analytics mediante la CLI de Azure

En este inicio rápido se usa la CLI de Azure para definir un trabajo de Stream Analytics que filtra mensajes del sensor en tiempo real con una lectura de la temperatura superior a 27. El trabajo de Stream Analytics lee los datos de un dispositivo de IoT Hub, los transforma y los escribe a su vez en un contenedor de Blob Storage. Un simulador en línea de Raspberry Pi genera los datos de entrada de esta guía de inicio rápido.

## <a name="before-you-begin"></a>Antes de empezar

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Cree un grupo de recursos. Todos los recursos de Azure se deben implementar en un grupo de recursos. Los grupos de recursos le permiten organizar y administrar los recursos relacionados de Azure.

   Para este inicio rápido, cree un grupo de recursos denominado *streamanalyticsrg* en la ubicación *eastus* con el comando [az group create](/cli/azure/group#az_group_create) siguiente:

   ```azurecli
   az group create --name streamanalyticsrg --location eastus
   ```

## <a name="prepare-the-input-data"></a>Preparación de los datos de entrada

Antes de definir el trabajo de Stream Analytics, prepare los datos que se usan para la entrada del trabajo.

Los siguientes bloques de código de la CLI de Azure son comandos que preparan los datos de entrada que requiere el trabajo. Examine las secciones para comprender el código.

1. Para crear un centro de IoT Hub, use el comando [az iot hub create](../iot-hub/iot-hub-create-using-cli.md#create-an-iot-hub). En este ejemplo se va a crear un centro de IoT Hub llamado **MyASAIoTHub**. Dado que los nombres de IoT Hub son únicos, tiene que asignarle su propio nombre de IoT Hub. Establezca la SKU en F1 para usar el nivel gratuito si está disponible con su suscripción. Si no es así, elija el siguiente nivel más bajo.

    ```azurecli
    az iot hub create --name "MyASAIoTHub" --resource-group streamanalyticsrg --sku S1
    ```

    Una vez creado el centro de IoT Hub, obtenga la cadena de conexión de IoT Hub mediante el comando [az iot hub show-connection-string](/cli/azure/iot/hub). Copie la cadena de conexión completa y guárdela para cuando agregue el centro de IoT Hub como entrada para el trabajo de Stream Analytics.

    ```azurecli
    az iot hub show-connection-string --hub-name "MyASAIoTHub"
    ```

2. Agregue un dispositivo a IoT Hub con el comando [az iothub device-identity create](../iot-hub/quickstart-send-telemetry-c.md#register-a-device). En este ejemplo se crea un dispositivo denominado **MyASAIoTDevice**.

    ```azurecli
    az iot hub device-identity create --hub-name "MyASAIoTHub" --device-id "MyASAIoTDevice"
    ```

3. Obtenga la cadena de conexión del dispositivo con el comando [az iot hub device-identity show-connection-string](/cli/azure/ext/azure-iot/iot/hub/device-identity#ext-azure-iot-az-iot-hub-device-identity-show-connection-string). Copie la cadena de conexión completa y guárdela para cuando cree el simulador de Raspberry Pi.

    ```azurecli
    az iot hub device-identity show-connection-string --hub-name "MyASAIoTHub" --device-id "MyASAIoTDevice" --output table
    ```

    **Ejemplo de salida:**

    ```output
    HostName=MyASAIoTHub.azure-devices.net;DeviceId=MyASAIoTDevice;SharedAccessKey=a2mnUsg52+NIgYudxYYUNXI67r0JmNubmfVafojG8=
    ```

## <a name="create-a-blob-storage-account"></a>Creación de una cuenta de almacenamiento de blobs

Los siguientes bloques de código de la CLI de Azure crean una cuenta de Blob Storage que se usa para la salida del trabajo. Examine las secciones para comprender el código.

1. Las cuentas de almacenamiento de uso general se crean con el comando [az storage account create](/cli/azure/storage/account). Las cuentas de almacenamiento de uso general se pueden para los cuatro servicios: blobs, archivos, tablas y colas.

   No olvide reemplazar los valores del marcador de posición entre corchetes angulares por sus propios valores:

   ```azurecli
   az storage account create \
       --name <storage-account> \
       --resource-group streamanalyticsrg \
       --location eastus \
       --sku Standard_ZRS \
       --encryption-services blob
   ```

2. Obtenga la clave de su cuenta de almacenamiento. Para ello, debe ejecutar el comando [az storage account keys list](/cli/azure/storage/account/keys). Guarde esta clave para usarla en el paso siguiente.

   ```azurecli
   az storage account keys list -g streamanalyticsrg -n <storage-account>
   ```

3. Cree un contenedor para almacenar blobs con el comando [az storage container create](/cli/azure/storage/container). Use la clave de la cuenta de almacenamiento para autorizar la operación de creación del contenedor. Para obtener más información sobre la autorización de operaciones de datos con la CLI de Azure, consulte [Autorización del acceso a los datos de blobs o colas con la CLI de Azure](../storage/blobs/authorize-data-operations-cli.md).

   ```azurecli
   az storage container create \
       --account-name <storage-account> \
       --name sample-container \
       --account-key <key>
       --auth-mode key
   ```

## <a name="create-a-stream-analytics-job"></a>Creación de un trabajo de Stream Analytics

Los siguientes bloques de código de la CLI de Azure crean un trabajo de Stream Analytics. Examine las secciones para comprender el código.

1. Cree un trabajo de Stream Analytics con el comando [az stream-analytics job create](/cli/azure/ext/stream-analytics/stream-analytics/job#ext-stream-analytics-az-stream-analytics-job-create).

```azurecli
az stream-analytics job create \
    --resource-group streamanalyticsrg 
    --name streamanalyticsjob \
    --location eastus \
    --output-error-policy "Drop" \
    --events-outoforder-policy "Drop" \
    --events-outoforder-max-delay 5 \
    --events-late-arrival-max-delay 16 \     
    --data-locale "en-US"
```

## <a name="configure-input-to-the-job"></a>Configuración de la entrada al trabajo

Agregue una entrada al trabajo mediante el cmdlet [az stream-analytics input](/cli/azure/ext/stream-analytics/stream-analytics/input#ext-stream-analytics-az-stream-analytics-input-create). Este cmdlet toma como parámetros el nombre del trabajo, el nombre de entrada del trabajo, el nombre del grupo de recursos y la definición de entrada del trabajo. La definición de la entrada del trabajo es un archivo JSON que contiene las propiedades necesarias para configurar la entrada del trabajo. En este ejemplo, creará una instancia de IoT Hub como entrada.

En su máquina local, cree un archivo denominado `datasource.json` y agréguele los siguientes datos JSON. No olvide reemplazar el valor de `sharedAccessPolicyKey` con la parte `SharedAccessKey` de la cadena de conexión de IoT Hub que guardó en una sección anterior.

```json
{
    "type": "Microsoft.Devices/IotHubs",
    "properties": {
        "iotHubNamespace": "iothub",
        "sharedAccessPolicyName": "iothubowner",
        "sharedAccessPolicyKey": "sharedAccessPolicyKey=",
        "consumerGroupName": "sdkconsumergroup",
        "endpoint": "messages/events"
    }
}
```

En su máquina local, cree un archivo denominado `serialization.json` y agréguele los siguientes datos JSON.

```json
{
     "type": "Json",
     "properties": {
         "encoding": "UTF8"
     }
}
```

Después, ejecute el cmdlet `az stream-analytics input create`. Asegúrese de reemplazar el valor de la variable `datasource` por la ruta de acceso a la ubicación en que almacenó el archivo JSON de definición de la entrada del trabajo y el valor de la variable `serialization` por la ruta de acceso en que ha almacenado el archivo JSON de serialización.

```azurecli
az stream-analytics input create \
    --resource-group streamanalyticsrg 
    --job-name streamanalyticsjob \
    --name asaiotinput \
    --type Stream \
    --datasource datasource.json \
    --serialization serialization.json
```

## <a name="configure-output-to-the-job"></a>Configuración de la salida al trabajo

Agregue una salida al trabajo mediante el cmdlet [az stream-analytics output create](/cli/azure/ext/stream-analytics/stream-analytics/output#ext-stream-analytics-az-stream-analytics-output-create). Este cmdlet toma como parámetros el nombre del trabajo, el nombre de salida del trabajo, el nombre del grupo de recursos y la definición de salida del trabajo. La definición de la salida del trabajo es un archivo JSON que contiene las propiedades necesarias para configurar la salida del trabajo. En este ejemplo se usa el almacenamiento de blobs como salida.

En su máquina local, cree un archivo denominado `datasink.json` y agréguele los siguientes datos JSON. Asegúrese de reemplazar el valor de `accountKey` por la clave de acceso de la cuenta de almacenamiento que sea el valor almacenado en $storageAccountKey.

```json
{
    "type": "Microsoft.Storage/Blob",
    "properties": {
        "storageAccounts": [
            {
                "accountName": "<storage-account>",
                "accountKey": "accountKey=="
            }
        ],
        "container": "state",
        "pathPattern": "{date}/{time}",
        "dateFormat": "yyyy/MM/dd",
        "timeFormat": "HH"
    }
}
```

Después, ejecute el cmdlet `az stream-analytics output`. Asegúrese de reemplazar el valor de la variable `datasource` por la ruta de acceso a la ubicación en que almacenó el archivo JSON de definición de la salida del trabajo y el valor de la variable `serialization` por la ruta de acceso en que ha almacenado el archivo JSON de serialización.

```azurecli
az stream-analytics output create \
    --resource-group streamanalyticsrg \
    --job-name streamanalyticsjob \
    --name asabloboutput \
    --datasource datasink.json \
    --serialization serialization.json
```

## <a name="define-the-transformation-query"></a>Definir la consulta de transformación

Agregue una transformación a su trabajo mediante el cmdlet [az stream-analytics transformation create](/cli/azure/ext/stream-analytics/stream-analytics/transformation#ext-stream-analytics-az-stream-analytics-transformation-create). Este cmdlet toma como parámetros el nombre del trabajo, el nombre de transformación del trabajo, el nombre del grupo de recursos y la definición de transformación del trabajo. 

Ejecute el cmdlet `az stream-analytics transformation create`.

```azurecli
az stream-analytics transformation create \
    --resource-group streamanalyticsrg \
    --job-name streamanalyticsjob \
    --name Transformation \
    --streaming-units "6" \
    --transformation-query "SELECT * INTO asabloboutput FROM asaiotinput HAVING Temperature > 27"
```
## <a name="run-the-iot-simulator"></a>Ejecutar el simulador

1. Abra el [simulador en línea de Raspberry Pi para Azure IoT](https://azure-samples.github.io/raspberry-pi-web-simulator/).

2. Reemplace el marcador de posición de la línea 15 con la cadena de conexión completa del dispositivo de Azure IoT Hub que guardó en la sección anterior.

3. Haga clic en **Ejecutar**. La salida debe mostrar los datos y mensajes del sensor que se envían a la instancia de IoT Hub.

    ![Simulador en línea de Raspberry Pi para Azure IoT](./media/stream-analytics-quick-create-powershell/ras-pi-connection-string.png)

## <a name="start-the-stream-analytics-job-and-check-the-output"></a>Inicio del trabajo de Stream Analytics y consulta de la salida

Inicie el trabajo mediante el cmdlet [az stream-analytics job start](/cli/azure/ext/stream-analytics/stream-analytics/job#ext-stream-analytics-az-stream-analytics-job-start). Este cmdlet toma como parámetros el nombre del trabajo, el nombre del grupo de recursos, el modo de inicio de salida y el tiempo de inicio. `OutputStartMode` acepta valores de `JobStartTime`, `CustomTime` o `LastOutputEventTime`.

Después de ejecutar el siguiente cmdlet, devuelve `True` como salida si el trabajo se inicia. En el contenedor de almacenamiento, se crea una carpeta de salida con los datos transformados.

```azurecli
az stream-analytics job start \
    --resource-group streamanalyticsrg \
    --name streamanalyticsjob \
    --output-start-mode JobStartTime
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando no los necesite, elimine el grupo de recursos, el trabajo de streaming y todos los recursos relacionados. La eliminación del trabajo evita la facturación de las unidades de streaming utilizadas por el trabajo. Si planea utilizar el trabajo en el futuro, puede omitir su eliminación y simplemente detenerlo por el momento. Si no va a seguir usando este trabajo, ejecute el siguiente cmdlet para eliminar todos los recursos creados en este inicio rápido:

```azurecli
az group delete \
    --name streamanalyticsrg \
    --no-wait
```

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha implementado un sencillo trabajo de Stream Analytics mediante la CLI de Azure. También puede implementar trabajos de Stream Analytics mediante [Azure Portal](stream-analytics-quick-create-portal.md) y [Visual Studio](stream-analytics-quick-create-vs.md).

Para aprender a configurar otros orígenes de entrada y realizar la detección en tiempo real, continúe con el siguiente artículo:

> [!div class="nextstepaction"]
> [Detección de fraudes en tiempo real con Azure Stream Analytics](stream-analytics-real-time-fraud-detection.md)