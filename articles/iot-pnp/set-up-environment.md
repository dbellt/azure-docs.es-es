---
title: 'Inicio rápido: Configuración de los recursos de IoT necesarios para IoT Plug and Play | Microsoft Docs'
description: 'Inicio rápido: Creación de una instancia de IoT Hub y otra del servicio Device Provisioning para usarlas con los inicios rápidos y tutoriales de IoT Plug and Play.'
author: dominicbetts
ms.author: dobett
ms.date: 08/11/2020
ms.topic: quickstart
ms.service: iot-pnp
services: iot-pnp
ms.openlocfilehash: 28eedc02adad479a20bc54fb8b342035f3d5cb15
ms.sourcegitcommit: f6b76df4c22f1c605682418f3f2385131512508d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108325776"
---
# <a name="quickstart-set-up-your-environment-for-the-iot-plug-and-play-quickstarts-and-tutorials"></a>Inicio rápido: Configuración del entorno para inicios rápidos y tutoriales de IoT Plug and Play

Antes de poder completar cualquiera de los tutoriales y las guías de inicio rápido de IoT Plug and Play, debe configurar un centro de IoT y el servicio Device Provisioning (DPS) en la suscripción de Azure. También necesitará copias locales de los archivos de modelo utilizados por las aplicaciones de ejemplo y la herramienta Azure IoT Explorer.

## <a name="prerequisites"></a>Prerrequisitos

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

[!INCLUDE [azure-cli-prepare-your-environment-h3](../../includes/azure-cli-prepare-your-environment-h3.md)]

## <a name="create-the-resources"></a>Creación de los recursos

Cree un grupo de recursos de Azure para los recursos:

```azurecli-interactive
az group create --name my-pnp-resourcegroup --location centralus
```

Cree un Centro de IoT. El siguiente comando usa el nombre `my-pnp-hub` como ejemplo del nombre del centro de IoT que se va a crear. Elija un nombre único para el centro de IoT que se usará en lugar de `my-pnp-hub`:

```azurecli-interactive
az iot hub create --name my-pnp-hub --resource-group my-pnp-resourcegroup --sku F1 --partition-count 2
```

Cree una instancia de DPS. El siguiente comando usa el nombre `my-pnp-dps` como ejemplo del nombre de la instancia de DPS que se va a crear. Elija un nombre único para la instancia de DPS que se usará en lugar de `my-pnp-dps`:

```azurecli-interactive
az iot dps create --name my-pnp-dps --resource-group my-pnp-resourcegroup
```

Para vincular la instancia de DPS a su centro de IoT, use los comandos siguientes. Reemplace `my-pnp-dps` y `my-pnp-hub` por los nombres únicos que eligió previamente:

```azurecli-interactive
hubConnectionString=$(az iot hub connection-string show -n my-pnp-hub --key primary --query connectionString -o tsv)
az iot dps linked-hub create --dps-name my-pnp-dps --resource-group my-pnp-resourcegroup --location centralus --connection-string $hubConnectionString
```

## <a name="retrieve-the-settings"></a>Recuperación de la configuración

En algunos tutoriales y guías de inicio rápido se usa la cadena de conexión del centro de IoT. También necesita la cadena de conexión al configurar la herramienta Azure IoT Explorer. Recupere la cadena de conexión y tome nota de ella ahora. Reemplace `my-pnp-hub` por el nombre único que ha elegido para el centro de IoT:

```azurecli-interactive
az iot hub connection-string show -n my-pnp-hub --key primary --query connectionString
```

La mayoría de las guías de inicio rápido y los tutoriales usan el *ámbito de identificador* de la configuración de DPS. Recupere el ámbito de identificador y tome nota de él ahora. Reemplace `my-pnp-dps` por el nombre único que ha elegido para la instancia de DPS:

```azurecli-interactive
az iot dps show --name my-pnp-dps --query properties.idScope
```

Todas las guías de inicio rápido y los tutoriales usan una inscripción de dispositivos de DPS. Use el comando siguiente para crear una *inscripción de dispositivo individual* `my-pnp-device` en la instancia de DPS. Reemplace `my-pnp-dps` por el nombre único que ha elegido para la instancia de DPS. Anote el identificador de registro y los valores de clave principal que se usarán en las guías de inicio rápido y los tutoriales:

```azurecli-interactive
az iot dps enrollment create --attestation-type symmetrickey --dps-name my-pnp-dps --resource-group my-pnp-resourcegroup --enrollment-id my-pnp-device --device-id my-pnp-device --query '{registrationID:registrationId,primaryKey:attestation.symmetricKey.primaryKey}'
```

## <a name="create-environment-variables"></a>Creación de variables de entorno

Cree cinco variables de entorno para configurar los ejemplos en las guías de inicio rápido y los tutoriales, a fin de usar el servicio Device Provisioning (DPS) para conectarse al centro de IoT:

* **IOTHUB_DEVICE_SECURITY_TYPE**: el valor `DPS`.
* **IOTHUB_DEVICE_DPS_ID_SCOPE**: el ámbito de identificador de DPS del que tomó nota anteriormente.
* **IOTHUB_DEVICE_DPS_DEVICE_ID**: el valor `my-pnp-device`.
* **IOTHUB_DEVICE_DPS_DEVICE_KEY**: clave principal de inscripción de la que tomó nota anteriormente.
* **IOTHUB_DEVICE_DPS_ENDPOINT**: el valor `global.azure-devices-provisioning.net`.

Los ejemplos del servicio necesitan las siguientes variables de entorno para identificar el centro y el dispositivo con los que se establecerá la conexión:

* **IOTHUB_CONNECTION_STRING**: la cadena de conexión de IoT Hub que anotó anteriormente.
* **IOTHUB_DEVICE_ID**: `my-pnp-device`.

Por ejemplo, en un shell con Bash de Linux:

```bash
export IOTHUB_DEVICE_SECURITY_TYPE="DPS"
export IOTHUB_DEVICE_DPS_ID_SCOPE="<Your ID scope>"
export IOTHUB_DEVICE_DPS_DEVICE_ID="my-pnp-device"
export IOTHUB_DEVICE_DPS_DEVICE_KEY="<Your enrolment primary key>"
export IOTHUB_DEVICE_DPS_ENDPOINT="global.azure-devices-provisioning.net"
export IOTHUB_CONNECTION_STRING="<Your IoT hub connection string>"
export IOTHUB_DEVICE_ID="my-pnp-device"
```

Por ejemplo, en la línea de comandos de Windows:

```cmd
set IOTHUB_DEVICE_SECURITY_TYPE=DPS
set IOTHUB_DEVICE_DPS_ID_SCOPE=<Your ID scope>
set IOTHUB_DEVICE_DPS_DEVICE_ID=my-pnp-device
set IOTHUB_DEVICE_DPS_DEVICE_KEY=<Your enrolment primary key>
set IOTHUB_DEVICE_DPS_ENDPOINT=global.azure-devices-provisioning.net
set IOTHUB_CONNECTION_STRING=<Your IoT hub connection string>
set IOTHUB_DEVICE_ID=my-pnp-device
```

## <a name="download-the-model-files"></a>Descarga de los archivos de modelo

En las guías de inicio rápido y los tutoriales se usan archivos de modelo de ejemplo para el controlador de temperatura y los dispositivos de termostato. Para descargar los archivos de modelo de ejemplo:

1. Cree una carpeta denominada *models* en la máquina local.

1. Haga clic con el botón derecho en [TemperatureController.json](https://raw.githubusercontent.com/Azure/opendigitaltwins-dtdl/master/DTDL/v2/samples/TemperatureController.json) y guarde el archivo JSON en la carpeta *models*.

1. Haga clic con el botón derecho en [Thermostat.json](https://raw.githubusercontent.com/Azure/opendigitaltwins-dtdl/master/DTDL/v2/samples/Thermostat.json) y guarde el archivo JSON en la carpeta *models*.

## <a name="install-the-azure-iot-explorer"></a>Instalación de Azure IoT Explorer

En las guías de inicio rápido y los tutoriales se usa la herramienta **Azure IoT Explorer**. Vaya a las [versiones de Azure IoT Explorer](https://github.com/Azure/azure-iot-explorer/releases) y expanda la lista de recursos de la versión más reciente. Descargue e instale la última versión de la aplicación para el sistema operativo.

La primera vez que ejecute la herramienta, se le solicitará la cadena de conexión del centro de IoT. Use la cadena de conexión que anotó anteriormente.

Configure la herramienta para usar los archivos de modelo que descargó anteriormente. En la página principal de la herramienta, seleccione **IoT Plug and Play Settings** (Configuración de IoT Plug and Play) y, a continuación, **+ Agregar > Carpeta local**. Seleccione la carpeta *models* que creó anteriormente. A continuación, seleccione **Guardar** para guardar la configuración.

Para obtener más información, consulte [Instalación y uso del explorador de Azure IoT](howto-use-iot-explorer.md).

## <a name="clean-up-resources"></a>Limpieza de recursos

Puede usar el centro de IoT y la instancia de DPS para todos los tutoriales y las guías de inicio rápido de IoT Plug and Play, por lo que solo necesita completar los pasos de este artículo una vez. Cuando haya terminado, puede quitarlos de la suscripción con el siguiente comando:

```azurecli-interactive
az group delete --name my-pnp-resourcegroup
```

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha configurado el entorno, puede probar una de las guías de inicio rápido o tutoriales, como:

> [!div class="nextstepaction"]
> [Conexión de una aplicación de dispositivo IoT Plug and Play de ejemplo a IoT Hub](quickstart-connect-device.md)
