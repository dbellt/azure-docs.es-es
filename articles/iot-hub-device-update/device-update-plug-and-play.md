---
title: Información sobre el modo en que Device Update para IoT Hub utiliza IoT Plug and Play | Microsoft Docs
description: Device Update para IoT Hub utiliza IoT Plug and Play para detectar y administrar dispositivos que admiten actualizaciones de forma inalámbrica.
author: ValOlson
ms.author: valls
ms.date: 2/14/2021
ms.topic: conceptual
ms.service: iot-hub-device-update
ms.openlocfilehash: fbc3502952e11830ef9abb06cb709fcc60288343
ms.sourcegitcommit: 425420fe14cf5265d3e7ff31d596be62542837fb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107739537"
---
# <a name="device-update-for-iot-hub-and-iot-plug-and-play"></a>Device Update para IoT Hub e IoT Plug and Play

Device Update para IoT Hub utiliza [IoT Plug and Play](../iot-pnp/index.yml) para detectar y administrar dispositivos que admiten actualizaciones de forma inalámbrica. El servicio Device Update enviará propiedades y mensajes a los dispositivos y los recibirá de estos utilizando interfaces de PnP. Device Update para IoT Hub necesita que los dispositivos IoT implementen las siguientes interfaces e identificadores de modelo que se indican a continuación.

## <a name="adu-core-interface"></a>Interfaz principal de ADU

La interfaz "ADUCoreInterface" se usa para enviar metadatos y acciones de actualización a los dispositivos y para recibir de ellos el estado de actualización. La interfaz principal de ADU se divide en dos propiedades de objeto.

Cuando se implementa esta interfaz, el nombre del componente que el modelo espera es **"azureDeviceUpdateAgent"** . [Más información sobre los componentes de PnP para Azure IoT](../iot-pnp/concepts-components.md)

### <a name="agent-metadata"></a>Metadatos del agente

Los metadatos del agente contienen campos que se utilizan en el dispositivo o en Device Update Agent para enviar información y el estado a los servicios de Device Update.

|Nombre|Schema|Dirección|Descripción|Ejemplo|
|----|------|---------|-----------|-----------|
|resultCode|integer|entre el dispositivo y la nube|Código que contiene información sobre el resultado de la última acción de actualización. Se puede especificar para indicar si se ha realizado correcta o incorrectamente y debe seguir la [especificación de códigos de estado HTTP](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html).|500|
|extendedResultCode|integer|entre el dispositivo y la nube|Código que contiene información adicional sobre el resultado. Se puede especificar para indicar si la operación se ha realizado correcta o incorrectamente.|0x80004005|
|state|integer|entre el dispositivo y la nube|Entero que indica el estado actual de Device Update Agent. Vea más abajo para obtener más información. |Inactivo|
|installedUpdateId|string|entre el dispositivo y la nube|Identificador de la actualización que está instalada actualmente (mediante Device Update). Este valor será null en los dispositivo que nunca han realizado una actualización mediante Device Update.|Null|
|`deviceProperties`|Asignación|entre el dispositivo y la nube|Conjunto de propiedades que contienen el fabricante y el modelo.|Vea más abajo para obtener más información.

#### <a name="state"></a>State

Es el estado que notifica Device Update Agent después de recibir una acción del servicio Device Update. `State` es la respuesta a una `Action` ([Acción], consulte `Actions` a continuación) que se notifica y se envía a Device Update Agent desde el servicio Device Update. Consulte el [flujo de trabajo general](understand-device-update.md#device-update-agent) de las solicitudes que se transmiten entre el servicio Device Update y Device Update Agent.

|Nombre|Value|Descripción|
|---------|-----|-----------|
|Inactivo|0|El dispositivo está listo para recibir una acción del servicio Device Update. Cuando una actualización se realiza correctamente, el estado vuelve a ser `Idle`.|
|DownloadSucceeded|2|Descarga correcta.|
|InstallSucceeded|4|Instalación correcta.|
|Con error|255|Se produjo un error durante la actualización.|

#### <a name="device-properties"></a>Propiedades de dispositivos

Conjunto de propiedades que contienen el fabricante y el modelo.

|Nombre|Schema|Dirección|Descripción|
|----|------|---------|-----------|
|fabricante|string|entre el dispositivo y la nube|Fabricante del dispositivo notificado mediante `deviceProperties`. Esta propiedad puede verse en uno de los dos lugares en los que la interfaz "AzureDeviceUpdateCore" va a intentar leer primero el valor "aduc_manufacturer" del [archivo de configuración](device-update-configuration-file.md).  Si el valor no figura en el archivo de configuración, se establecerá de forma predeterminada en un valor que informe del tiempo de compilación de ADUC_DEVICEPROPERTIES_MANUFACTURER. Esta propiedad solo se notificará en el momento del arranque.|
|model|string|entre el dispositivo y la nube|Modelo del dispositivo, que se notifica mediante `deviceProperties`. Esta propiedad puede verse en uno de los dos lugares en los que la interfaz "AzureDeviceUpdateCore" va a intentar leer primero el valor de "aduc_model" del [archivo de configuración](device-update-configuration-file.md).  Si el valor no figura en el archivo de configuración, se establecerá de forma predeterminada en un valor que informe del tiempo de compilación de ADUC_DEVICEPROPERTIES_MODEL. Esta propiedad solo se notificará en el momento del arranque.|
|aduVer|string|entre el dispositivo y la nube|Versión de Device Update Agent que se ejecuta en el dispositivo. Este valor solamente se obtiene de la compilación si durante el tiempo de compilación ENABLE_ADU_TELEMETRY_REPORTING está establecido en 1 (true). Los clientes pueden desactivar la notificación de la versión estableciendo el valor en 0 (false). [Personalización de las propiedades de Device Update Agent](https://github.com/Azure/iot-hub-device-update/blob/main/docs/agent-reference/how-to-build-agent-code.md).|
|doVer|string|entre el dispositivo y la nube|Versión del agente de Optimización de distribución que se ejecuta en el dispositivo. Este valor solamente se obtiene de la compilación si durante el tiempo de compilación ENABLE_ADU_TELEMETRY_REPORTING está establecido en 1 (true). Los clientes pueden desactivar la notificación de la versión estableciendo el valor en 0 (false). [Personalización de las propiedades de Device Update Agent](https://github.com/microsoft/do-client/blob/main/README.md#building-do-client-components).|

### <a name="service-metadata"></a>Metadatos de servicio

Los metadatos del servicio contienen campos que los servicios de Device Update utilizan para notificar acciones y datos a Device Update Agent.

|Nombre|Schema|Dirección|Descripción|
|----|------|---------|-----------|
|action|integer|entre la nube y el dispositivo|Entero que se corresponde con una acción que el agente debe realizar. A continuación, se muestran los valores.|
|updateManifest|string|entre la nube y el dispositivo|Se usa para describir el contenido de una actualización. Se genera a partir del [manifiesto de importación](import-update.md#create-a-device-update-import-manifest)|
|updateManifestSignature|Objeto JSON|entre la nube y el dispositivo|JSON Web Signature (JWS, firma web de JSON) con las JSON Web Keys (claves web de JSON) que se utilizan para comprobar el código fuente.|
|fileUrls|Asignación|entre la nube y el dispositivo|Asignación de `FileHash` a `DownloadUri`. Indica al agente qué archivos se van a descargar y el hash que se va a usar para comprobar que los archivos se descargaron correctamente.|

#### <a name="action"></a>Acción

El objeto `Actions` siguiente representa las acciones realizadas por Device Update Agent conforme a las instrucciones del servicio Device Update. Device Update Agent notificará un objeto `State` (consulte `State` en la sección anterior) sobre el procesamiento del valor de `Action` recibido. Consulte el [flujo de trabajo general](understand-device-update.md#device-update-agent) de las solicitudes que se transmiten entre el servicio Device Update y Device Update Agent.

|Nombre|Value|Descripción|
|---------|-----|-----------|
|Descargar|0|Descarga la actualización o el contenido que están publicados y cualquier otro contenido necesario|
|Instalar|1|Instala el contenido o la actualización. Normalmente, esto significa que es necesario llamar al instalador del contenido o la actualización.|
|Aplicar|2|Finaliza la actualización. Indica al sistema que debe reiniciarse, si es necesario.|
|Cancelar|255|Detiene el procesamiento de la acción actual y vuelve a `Idle`. También se usará para indicar al agente con el estado `Failed` que vuelva a `Idle`.|

## <a name="device-information-interface"></a>Interfaz de información del dispositivo

La interfaz de información del dispositivo es un concepto que se utiliza en la [arquitectura de IoT Plug and Play](../iot-pnp/overview-iot-plug-and-play.md). Contiene las propiedades entre el dispositivo y la nube que proporcionan información sobre el hardware y el sistema operativo del dispositivo. Device Update for IoT Hub utiliza las propiedades DeviceInformation.manufacturer y DeviceInformation.model para telemetría y diagnóstico. Para más información sobre la interfaz de información del dispositivo, consulte este [ejemplo](https://devicemodels.azure.com/dtmi/azure/devicemanagement/deviceinformation-1.json).

Cuando se implementa esta interfaz, el nombre que el modelo espera para el componente es **deviceInformation**. [Más información sobre los componentes de Azure IoT PnP](../iot-pnp/concepts-components.md)

|Nombre|Tipo|Schema|Dirección|Descripción|Ejemplo|
|----|----|------|---------|-----------|-----------|
|fabricante|Propiedad|string|entre el dispositivo y la nube|Nombre de la compañía del fabricante del dispositivo. Puede ser igual que el nombre del fabricante de equipos originales (OEM).|Contoso|
|model|Propiedad|string|entre el dispositivo y la nube|Nombre o identificador del modelo del dispositivo.|Dispositivo de IoT Edge|
|swVersion|Propiedad|string|entre el dispositivo y la nube|Versión del software del dispositivo. El valor de swVersion podría ser la versión del firmware.|4.15.0-122|
|osName|Propiedad|string|entre el dispositivo y la nube|Nombre del sistema operativo del dispositivo.|Ubuntu Server 18.04|
|processorArchitecture|Propiedad|string|entre el dispositivo y la nube|Arquitectura del procesador del dispositivo.|ARM64|
|processorManufacturer|Propiedad|string|entre el dispositivo y la nube|Nombre del fabricante del procesador del dispositivo.|Microsoft|
|totalStorage|Propiedad|string|entre el dispositivo y la nube|Almacenamiento total en kilobytes disponible en el dispositivo.|2048|
|totalMemory|Propiedad|string|entre el dispositivo y la nube|Memoria total en kilobytes disponible en el dispositivo.|256|

## <a name="model-id"></a>Id. de modelo 

El identificador del modelo es la forma en que Smart Devices anuncia sus funcionalidades a las aplicaciones de IoT de Azure con IoT plug and Play. Si necesita más información sobre la creación de Smart Devices para anunciar sus funcionalidades en aplicaciones de Azure IoT, visite la [guía para desarrolladores de dispositivos IoT Plug and Play](../iot-pnp/concepts-developer-guide-device.md).

Device Update para IoT Hub necesita que los Smart Devics de IoT Plug and Play anuncien un identificador de modelo con el valor **"dtmi:AzureDeviceUpdate;1"** durante la conexión del dispositivo. [Descubra cómo se anuncia un identificador de modelo](../iot-pnp/concepts-developer-guide-device.md#model-id-announcement).