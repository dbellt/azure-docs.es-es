---
title: Implementación del puente de dispositivos de Azure IoT Central | Microsoft Docs
description: Implementación del puente de dispositivos de IoT Central para conectar otras nubes de IoT a la aplicación de IoT Central. Otras nubes de IoT incluyen Sigfox, Particle Device Cloud y The Things Network.
services: iot-central
ms.service: iot-central
author: dominicbetts
ms.author: dobett
ms.date: 04/19/2021
ms.topic: how-to
ms.openlocfilehash: f32f36399eec2b54b872ae9750ea2ddf2fb5a218
ms.sourcegitcommit: b35c7f3e7f0e30d337db382abb7c11a69723997e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109683792"
---
# <a name="use-the-iot-central-device-bridge-to-connect-other-iot-clouds-to-iot-central"></a>Uso del puente de dispositivos de IoT Central para conectar otras nubes de IoT a IoT Central

El puente de dispositivos de IoT Central es una solución de código abierto que conecta otras nubes de IoT a su aplicación de IoT Central. Entre los ejemplos de otras nubes de IoT se incluyen [Sigfox](https://www.sigfox.com/), [Device Cloud de Particle](https://www.particle.io/) y [The Things Network](https://www.thethingsnetwork.org/). El puente de dispositivos funciona mediante el reenvío de datos desde dispositivos conectados a otras nubes de IoT a través de la aplicación de IoT Central. El puente de dispositivo solo reenvía datos a IoT Central, no envía comandos ni actualizaciones de propiedades de IoT Central a los dispositivos.

El puente de dispositivos permite combinar el potencial de IoT Central con dispositivos como dispositivos de seguimiento de recursos conectados a la red de área amplia de baja potencia de Sigfox, dispositivos de control de la calidad del aire en Device Cloud de Particle o dispositivos de supervisión de la humedad de suelo de The Things Network. Puede usar características de la aplicación de IoT Central como reglas y análisis en los datos, crear flujos de trabajo en Power Automate y Azure Logic Apps, o exportar los datos.

La solución del puente de dispositivos aprovisiona varios recursos de Azure en la suscripción de Azure que funcionan conjuntamente para transformar y reenviar mensajes del dispositivo a IoT Central.

## <a name="prerequisites"></a>Requisitos previos

Necesitará lo siguiente para completar los pasos de esta guía:

[!INCLUDE [iot-central-prerequisites-basic](../../../includes/iot-central-prerequisites-basic.md)]

## <a name="overview"></a>Información general

El puente de dispositivos de IoT Central es una solución de código abierto en GitHub. Usa una plantilla de Azure Resource Manager personalizada para implementar varios recursos en la suscripción de Azure, incluida una aplicación de funciones de Azure Functions.

La aplicación de funciones es la pieza fundamental del puente de dispositivos. Recibe las solicitudes HTTP POST de otras plataformas de IoT a través de un webhook simple. El [puente de dispositivos de Azure IoT Central](https://github.com/Azure/iotc-device-bridge) incluye ejemplos que muestran cómo conectar nubes de Sigfox, Particle y The Things Network. Puede ampliar esta solución para conectarse a su nube de IoT personalizada si su plataforma puede enviar solicitudes HTTP POST a la aplicación de funciones.

La aplicación de funciones transforma los datos en un formato aceptado por IoT Central y los reenvía mediante el servicio de aprovisionamiento de dispositivos y las API de cliente del dispositivo:

:::image type="content" source="media/howto-build-iotc-device-bridge/azure-function.png" alt-text="Captura de pantalla de Azure Functions.":::

Si la aplicación de IoT Central reconoce el identificador de dispositivo en el mensaje reenviado, la telemetría del dispositivo aparece en IoT Central. Si la aplicación de IoT Central no reconoce el identificador de dispositivo, la aplicación de funciones intenta registrar un nuevo dispositivo con el identificador de dispositivo. El nuevo dispositivo aparece como **un dispositivo no asociado** en la página **Dispositivos** de la aplicación de IoT Central. En la página **Dispositivos**, puede asociar el nuevo dispositivo a una plantilla de dispositivo y, a continuación, ver la telemetría.

## <a name="deploy-the-device-bridge"></a>Implementación del puente de dispositivos

Para implementar el puente de dispositivos en su suscripción:

1. En la aplicación de IoT Central, vaya a la página **Administration > Device Connection** (Administración > Conexión de dispositivos).

    1. Anote el **ámbito de identificador**. Este valor se usa al implementar el puente de dispositivos.

    1. En la misma página, abra el grupo de inscripción **SAS-IoT-Devices**. En la página del grupo **SAS-IoT-Devices**, copie la **clave principal**. Este valor se usa al implementar el puente de dispositivos.

1. Use el botón **Implementar en Azure** siguiente para abrir la plantilla de Resource Manager personalizada que implementa la aplicación de funciones en su suscripción. Use el **ámbito de identificador** y la **clave principal** del paso anterior:

    [![Implementar en Azure](http://azuredeploy.net/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fiotc-device-bridge%2Fmaster%2Fazuredeploy.json)

Una vez completada la implementación, debe instalar los paquetes NPM que requiere la función:

1. En Azure Portal, abra la aplicación de funciones que se implementó en su suscripción. Luego vaya a **Herramientas de desarrollo** > **Consola**. En la consola, ejecute los siguientes comandos para instalar los paquetes:

    ```bash
    cd IoTCIntegration
    npm install
    ```

    La ejecución de estos comandos pueden llevar varios minutos. Puede ignorar los mensajes de advertencia de forma segura.

1. Una vez finalizada la instalación del paquete, seleccione **Reiniciar** en la página **Información general** de la aplicación de funciones:

    :::image type="content" source="media/howto-build-iotc-device-bridge/restart.png" alt-text="Captura de pantalla de Reiniciar":::

1. La función ya está lista para su uso. Los sistemas externos pueden usar solicitudes HTTP POST para enviar datos del dispositivo a través del puente de dispositivos a su aplicación de IoT Central. Para obtener la dirección URL de la función, vaya a **Functions > IoTCIntegration > Código + prueba > Obtener la dirección URL de la función**:

    :::image type="content" source="media/howto-build-iotc-device-bridge/get-function-url.png" alt-text="Captura de pantalla de Obtener la dirección URL de la función.":::

Los cuerpos de mensajes enviados al puente de dispositivos deben tener el formato siguiente:

```json
"device": {
  "deviceId": "my-cloud-device"
},
"measurements": {
  "temp": 20.31,
  "pressure": 50,
  "humidity": 8.5,
  "ledColor": "blue"
}
```

Cada clave del objeto `measurements` debe coincidir con el nombre de un tipo de telemetría en la plantilla de dispositivo de la aplicación de IoT Central. Esta solución no admite la especificación del identificador de interfaz en el cuerpo del mensaje. Por lo tanto, si dos interfaces diferentes tienen un tipo de telemetría con el mismo nombre, la medida aparece en los dos flujos de telemetría de la aplicación de IoT Central.

Puede incluir un campo `timestamp` en el cuerpo para especificar la fecha y hora UTC del mensaje. Este campo debe estar en formato ISO 8601. Por ejemplo, `2020-06-08T20:16:54.602Z`. Si no incluye una marca de tiempo, se usan la fecha y hora actuales.

Puede incluir un campo `modelId` en el cuerpo. Use este campo para asociar el dispositivo a una plantilla de dispositivo durante el aprovisionamiento. Esta funcionalidad solo es compatible con las [aplicaciones V3](howto-get-app-info.md).

`deviceId` debe estar formado por caracteres alfanuméricos y en minúscula, y puede contener guiones.

Si no incluye el campo `modelId` o si IoT Central no reconoce el identificador de modelo, un mensaje con un identificador `deviceId` no reconocido crea un nuevo _dispositivo no asociado_ en IoT Central. Un operador puede migrar manualmente el dispositivo a la plantilla de dispositivo correcta. Para obtener más información, consulte [Administración de dispositivos en la aplicación de Azure IoT Central > Migración de dispositivos a una plantilla](howto-manage-devices.md).

En las [aplicaciones V2](howto-get-app-info.md), el nuevo dispositivo aparece en la página **Explorador de dispositivos > Dispositivos no asociados**. Seleccione **Asociar** y elija una plantilla de dispositivo para empezar a recibir telemetría entrante del dispositivo.

> [!NOTE]
> Hasta que el dispositivo esté asociado a una plantilla, todas las llamadas HTTP a la función devolverán un estado de error 403.

Para activar el registro de la aplicación de funciones con Application Insights, vaya a **Supervisión > Registros** en la aplicación de funciones en Azure Portal. Seleccione **Activar Application Insights**.

## <a name="provisioned-resources"></a>Recursos aprovisionados

La plantilla de Resource Manager aprovisiona los siguientes recursos en la suscripción de Azure:

* Aplicación de función
* Plan de App Service
* Cuenta de almacenamiento
* Almacén de claves

El almacén de claves almacena la clave de grupo SAS para la aplicación de IoT Central.

La aplicación de funciones se ejecuta en un [plan de consumo](https://azure.microsoft.com/pricing/details/functions/). Aunque esta opción no ofrece recursos de proceso dedicados, permite que el puente de dispositivos controle cientos de mensajes de dispositivo por minuto, adecuados para flotas más pequeñas de dispositivos o dispositivos que envían mensajes con menos frecuencia. Si la aplicación depende de la transmisión de un gran número de mensajes del dispositivo, reemplace el plan de consumo por un[plan de App Service](https://azure.microsoft.com/pricing/details/app-service/windows/) dedicado. Este plan ofrece recursos de proceso dedicados, que ofrecen tiempos de respuesta del servidor más rápidos. Con un plan de App Service estándar, el rendimiento máximo observado de la función de Azure en este repositorio fue de aproximadamente 1500 mensajes de dispositivo por minuto. Para más información, consulte las [opciones de hospedaje de Azure Functions](../../azure-functions/functions-scale.md).

Para usar un plan de App Service dedicado en lugar de un plan de consumo, edite la plantilla personalizada antes de implementar. Seleccione **Editar plantilla**.

:::image type="content" source="media/howto-build-iotc-device-bridge/edit-template.png" alt-text="Captura de pantalla de Editar plantilla.":::

Reemplace el siguiente segmento:

```json
{
  "type": "Microsoft.Web/serverfarms",
  "apiVersion": "2015-04-01",
  "name": "[variables('planName')]",
  "location": "[resourceGroup().location]",
  "properties": {
    "name": "[variables('planName')]",
    "computeMode": "Dynamic",
    "sku": "Dynamic"
  }
},
```

con

```json
{
  "type": "Microsoft.Web/serverfarms",
  "sku": {
      "name": "S1",
      "tier": "Standard",
      "size": "S1",
      "family": "S",
      "capacity": 1
  },
  "kind": "app",
  "name": "[variables('planName')]",
  "apiVersion": "2016-09-01",
  "location": "[resourceGroup().location]",
  "tags": {
      "iotCentral": "device-bridge",
      "iotCentralDeviceBridge": "app-service-plan"
  },
  "properties": {
      "name": "[variables('planName')]"
  }
},
```

A continuación, edite la plantilla para incluir `"alwaysOn": true` en la configuración del recurso `functionapp` en `"properties": {"SiteConfig": {...}}` La [configuración de AlwaysOn](https://github.com/Azure/Azure-Functions/wiki/Enable-Always-On-when-running-on-dedicated-App-Service-Plan) garantiza que la aplicación de funciones se ejecute siempre.

## <a name="examples"></a>Ejemplos

En los ejemplos siguientes se describe cómo configurar el puente de dispositivos para varias nubes de IoT:

### <a name="example-1-connecting-particle-devices-through-the-device-bridge"></a>Ejemplo 1: Conexión de dispositivos Particle a través del puente de dispositivos

Para conectar un dispositivo Particle a través del puente de dispositivos a IoT Central, vaya a la consola de Particle y cree una nueva integración de webhook. Establezca el **formato de solicitud** en **JSON**.  En **Configuración avanzada**, use el siguiente formato de cuerpo personalizado:

```json
{
  "device": {
    "deviceId": "{{{PARTICLE_DEVICE_ID}}}"
  },
  "measurements": {
    "{{{PARTICLE_EVENT_NAME}}}": "{{{PARTICLE_EVENT_VALUE}}}"
  }
}
```

Pegue la **URL de función** de la aplicación de funciones; se ve que los dispositivos Particle aparecen como dispositivos no asociados en IoT Central. Para obtener más información, consulte la entrada de blog [Here's how to integrate your Particle-powered projects with Azure IoT Central](https://blog.particle.io/2019/09/26/integrate-particle-with-azure-iot-central/) (A continuación se indica cómo integrar los proyectos con tecnología de Particle con Azure IoT Central).

### <a name="example-2-connecting-sigfox-devices-through-the-device-bridge"></a>Ejemplo 2: Conexión de dispositivos Sigfox a través del puente de dispositivos

Es posible que algunas plataformas no le permitan especificar el formato de los mensajes de dispositivo enviados a través de un webhook. Para estos sistemas, debe convertir la carga del mensaje al formato de cuerpo esperado antes de que el puente de dispositivos la procese. Puede realizar la conversión en la misma función que ejecuta el puente de dispositivos.

En esta sección se muestra cómo convertir la carga de una integración de webhook de Sigfox al formato de cuerpo esperado por el puente de dispositivos. La nube de Sigfox transmite los datos del dispositivo en un formato de cadena hexadecimal. Para mayor comodidad, el puente de dispositivos incluye una función de conversión para este formato, que acepta un subconjunto de los posibles tipos de campo en una carga del dispositivo Sigfox:`int` y `uint` de 8, 16, 32 o 64 bits; `float` de 32 bits o 64 bits; little-endian y big-endian. Para procesar mensajes desde una integración de webhook de Sigfox, realice los siguientes cambios en el archivo _IoTCIntegration/index.js_ en la aplicación de funciones.

Para convertir la carga del mensaje, agregue el código siguiente antes de la llamada a `handleMessage` en la línea 21, reemplazando`payloadDefinition` por su definición de carga Sigfox:

```javascript
const payloadDefinition = 'gforce::uint:8 lat::uint:8 lon::uint:16'; // Replace this with your payload definition

req.body = {
    device: {
        deviceId: req.body.device
    },
    measurements: require('./converters/sigfox')(payloadDefinition, req.body.data)
};
```

Los dispositivos Sigfox esperan un código de respuesta `204`. Agregue el siguiente código después de la llamada a `handleMessage` en la línea 21:

```javascript
context.res = {
    status: 204
};
```

### <a name="example-3-connecting-devices-from-the-things-network-through-the-device-bridge"></a>Ejemplo 3: Conexión de dispositivos desde The Things Network a través del puente de dispositivos

Para conectar los dispositivos The Things Network a IoT Central:

* Agregue una nueva integración HTTP a su aplicación en The Things Network: **Aplicación > Integraciones > agregar integración > Integración HTTP**.
* Asegúrese de que la aplicación incluye una función descodificador que convierte automáticamente la carga de los mensajes del dispositivo en JSON antes de enviarla a la función: **Aplicación > Funciones de carga > descodificador**.

En el ejemplo siguiente se muestra una función descodificador de JavaScript que puede usar para descodificar tipos numéricos comunes a partir de datos binarios:

```javascript
function Decoder(bytes, port) {
  function bytesToFloat(bytes, decimalPlaces) {
    var bits = (bytes[3] << 24) | (bytes[2] << 16) | (bytes[1] << 8) | bytes[0];
    var sign = (bits >>> 31 === 0) ? 1.0 : -1.0;
    var e = bits >>> 23 & 0xff;
    var m = (e === 0) ? (bits & 0x7fffff) << 1 : (bits & 0x7fffff) | 0x800000;
    var f = Math.round((sign * m * Math.pow(2, e - 150)) * Math.pow(10, decimalPlaces)) / Math.pow(10, decimalPlaces);
    return f;
  }

  function bytesToInt32(bytes, signed) {
    var bits = bytes[0] | (bytes[1] << 8) | (bytes[2] << 16) | (bytes[3] << 24);
    var sign = 1;

    if (signed && bits >>> 31 === 1) {
      sign = -1;
      bits = bits & 0x7FFFFFFF;
    }

    return bits * sign;
  }

  function bytesToShort(bytes, signed) {
    var bits = bytes[0] | (bytes[1] << 8);
    var sign = 1;

    if (signed && bits >>> 15 === 1) {
      sign = -1;
      bits = bits & 0x7FFF;
    }

    return bits * sign;
  }

  return {
    temperature: bytesToFloat(bytes.slice(0, 4), 2),
    presscounter: bytesToInt32(bytes.slice(4, 8), true),
    blueLux: bytesToShort(bytes.slice(8, 10), false)
  };
}
```

Después de definir la integración, agregue el código siguiente antes de llamar a `handleMessage` en la línea 21 del archivo *IoTCIntegration/index.js* de la aplicación de funciones. Este código traduce el cuerpo de la integración HTTP al formato esperado.

```javascript
device: {
    deviceId: req.body.hardware_serial.toLowerCase()
},
measurements: req.body.payload_fields
};
```

## <a name="limitations"></a>Limitaciones

El puente de dispositivos solo reenvía mensajes a IoT Central y no envía mensajes a los dispositivos. Por eso, las propiedades y los comandos no funcionan en los dispositivos que se conectan a IoT Central a través de este puente de dispositivos. Dado que no se admiten operaciones de dispositivo gemelo, no es posible actualizar las propiedades del dispositivo a través del puente de dispositivos. Para usar estas características, un dispositivo debe conectarse directamente a IoT Central mediante uno de los [SDK de dispositivo de Azure IoT](../../iot-hub/iot-hub-devguide-sdks.md).

## <a name="next-steps"></a>Pasos siguientes
Ahora que ha aprendido a implementar el puente de dispositivos de IoT Central, este es el siguiente paso que se recomienda:

> [!div class="nextstepaction"]
> [Administración de dispositivos](howto-manage-devices.md)
