---
title: Transformación de datos para Azure IoT Central | Microsoft Docs
description: Los dispositivos de IoT envían datos en varios formatos que puede que necesite transformar. En este artículo se describe cómo transformar los datos tanto al realizar la transición a IoT Central como en la salida. Igualmente, los escenarios descritos usan IoT Edge y Azure Functions.
author: dominicbetts
ms.author: dobett
ms.date: 04/09/2021
ms.topic: how-to
ms.service: iot-central
services: iot-central
ms.openlocfilehash: 87f188b414791636e147a19202b6fcf314d6f307
ms.sourcegitcommit: b35c7f3e7f0e30d337db382abb7c11a69723997e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109683662"
---
# <a name="transform-data-for-iot-central"></a>Transformación de datos para IoT Central

Los dispositivos IoT envían datos en varios formatos. Para usar los datos del dispositivo con la aplicación IoT Central, es posible que tenga que usar una transformación para:

- Hacer que el formato de los datos sea compatible con la aplicación IoT Central.
- Convertir unidades.
- Procesar métricas nuevas.
- Enriquecer los datos de otros orígenes.

En este artículo se muestra cómo transformar los datos del dispositivo fuera de IoT Central, ya sea a la entrada o la salida.

En el diagrama siguiente se muestran tres rutas para los datos que incluyen transformaciones:

:::image type="content" source="media/howto-transform-data/transform-data.png" alt-text="Resumen de las rutas de transformación de datos tanto de entrada como de salida" border="false":::

En la tabla siguiente se muestran tres tipos de transformaciones de ejemplo:

| Transformación | Descripción | Ejemplo  | Notas |
|------------------------|-------------|----------|-------|
| Formato de los mensajes         | Convierta o manipule mensajes JSON. | CSV a JSON  | En la entrada. IoT Central solo acepta mensajes JSON de valor. Para más información, consulte [Cargas de telemetría, propiedades y comandos](concepts-telemetry-properties-commands.md). |
| Cálculos           | Funciones matemáticas que puede ejecutar [Azure Functions](../../azure-functions/index.yml). | Conversión de unidades de Fahrenheit a Celsius.  | Realice la transformación mediante el patrón de salida para aprovechar las ventajas de la entrada de dispositivo escalable a través de la conexión directa a IoT Central. La transformación de los datos le permite usar las características de IoT Central como visualizaciones y trabajos. |
| Enriquecimiento de mensajes     | Los enriquecimientos de orígenes de datos externos no se encuentran en las propiedades o la telemetría del dispositivo. Para obtener más información sobre los enriquecimientos internos, consulte [Exportación de datos de IoT a destinos en la nube mediante la exportación de datos](howto-export-data.md). | Agregue información meteorológica a los mensajes mediante los datos de ubicación de los dispositivos. | Realice la transformación mediante el patrón de salida para aprovechar las ventajas de la entrada de dispositivo escalable a través de la conexión directa a IoT Central. |

## <a name="prerequisites"></a>Requisitos previos

Necesitará lo siguiente para completar los pasos de esta guía:

[!INCLUDE [iot-central-prerequisites-basic](../../../includes/iot-central-prerequisites-basic.md)]

## <a name="data-transformation-at-ingress"></a>Transformación de datos en la entrada

Para transformar los datos del dispositivo en la entrada, hay dos opciones:

- **IoT Edge:** use un módulo de IoT Edge para transformar los datos de los dispositivos de bajada antes de enviar los datos a la aplicación IoT Central.

- **puente de dispositivos de IoT Central**: el [puente de dispositivos de IoT Central](howto-build-iotc-device-bridge.md) conecta otras nubes de dispositivos de IoT, como Sigfox, Particle y The Things Network, a IoT Central. El puente de dispositivos usa una función de Azure para reenviar los datos y puede personalizar esta función para transformar los datos del dispositivo.

### <a name="use-iot-edge-to-transform-device-data"></a>Uso de IoT Edge para transformar los datos del dispositivo

:::image type="content" source="media/howto-transform-data/transform-ingress.png" alt-text="Transformación de datos en la entrada mediante IoT Edge" border="false":::

En este escenario, un módulo de IoT Edge transforma los datos de los dispositivos de bajada antes de reenviarlos a la aplicación IoT Central. Estos son los pasos de alto nivel que se deben realizar para configurar este escenario:

1. **Configuración de un dispositivo de IoT Edge:** instale y aprovisione un dispositivo de IoT Edge como puerta de enlace y conéctela a la aplicación IoT Central.

1. **Conexión del dispositivo de bajada al dispositivo de IoT Edge:** conecte los dispositivos de bajada al dispositivo de IoT Edge y aprovisiónelos a la aplicación IoT Central.

1. **Transformación de los datos del dispositivo en IoT Edge:** cree un módulo de IoT Edge para transformar los datos. Implemente el módulo en el dispositivo de puerta de enlace IoT Edge que reenvía los datos del dispositivo transformados a la aplicación IoT Central.

1. **Comprobación:** envíe datos desde un dispositivo de bajada a la puerta de enlace y compruebe que los datos del dispositivo transformados llegan a la aplicación IoT Central.

En el ejemplo descrito en las secciones siguientes, el dispositivo de bajada envía datos CSV con el formato siguiente al dispositivo de puerta de enlace IoT Edge:

```csv
"<temperature >, <pressure>, <humidity>"
```

Quiere usar un módulo de IoT Edge para transformar los datos al siguiente formato JSON antes de enviarlos a IoT Central:

```json
{
  "device": {
      "deviceId": "<downstream-deviceid>"
  },
  "measurements": {
    "temp": <temperature>,
    "pressure": <pressure>,
    "humidity": <humidity>,
  }
}
```

[![Examinar el código](media/common/browse-code.svg)](https://github.com/iot-for-all/iot-central-transform-with-iot-edge)

En los pasos siguientes se muestra cómo instalar y configurar este escenario:

### <a name="build-the-custom-module"></a>Compilación del módulo personalizado

En este escenario, el dispositivo de IoT Edge ejecuta un módulo personalizado que transforma los datos del dispositivo de bajada. Antes de implementar y configurar el dispositivo de IoT Edge, debe:

- Compilar el módulo personalizado.
- Agregar el módulo personalizado a un registro de contenedor.

El entorno de ejecución de IoT Edge descarga módulos personalizados de un registro de contenedor, como un registro de contenedor de Azure o Docker Hub. La instancia de [Azure Cloud Shell](../../cloud-shell/overview.md) tiene todas las herramientas necesarias para crear un registro de contenedor, compilar el módulo y cargarlo en el registro:

Para crear un registro de contenedor:

1. Abra [Azure Cloud Shell](https://shell.azure.com/) e inicie sesión en su suscripción de Azure.

1. Ejecute los siguientes comandos para crear un registro de contenedor de Azure.

    ```azurecli
    REGISTRY_NAME="{your unique container registry name}"
    az group create --name ingress-scenario --location eastus
    az acr create -n $REGISTRY_NAME -g ingress-scenario --sku Standard --admin-enabled true
    az acr credential show -n $REGISTRY_NAME
    ```

    Tome nota de los valores `username` y `password`, puesto que los usará más adelante.

Para compilar el módulo personalizado en [Azure Cloud Shell](https://shell.azure.com/):

1. En [Azure Cloud Shell](https://shell.azure.com/), vaya a una carpeta adecuada.
1. Para clonar el repositorio de GitHub que contiene el código fuente del módulo, ejecute el siguiente comando:

    ```azurecli
    git clone https://github.com/iot-for-all/iot-central-transform-with-iot-edge
    ```

1. Para compilar el módulo personalizado, ejecute los siguientes comandos en Azure Cloud Shell:

    ```azurecli
    cd iot-central-transform-with-iot-edge/custommodule/transformmodule
    az acr build --registry $REGISTRY_NAME --image transformmodule:0.0.1-amd64 -f Dockerfile.amd64 .
    ```

    La ejecución de estos comandos puede llevar varios minutos.

### <a name="set-up-an-iot-edge-device"></a>Configuración de un dispositivo de IoT Edge

En este escenario se usa un dispositivo de puerta de enlace IoT Edge para transformar los datos de cualquier dispositivo de bajada. En esta sección se describe cómo crear plantillas de dispositivos de IoT Central para la puerta de enlace y los dispositivos de bajada en la aplicación IoT Central. Los dispositivos de IoT Edge usan un manifiesto de implementación para configurar sus módulos.

Para crear una plantilla de dispositivo para el dispositivo de bajada, en este escenario se usa un modelo de un termostato simple:

1. Descargue el [modelo de dispositivo para el termostato](https://raw.githubusercontent.com/Azure/iot-plugandplay-models/main/dtmi/com/example/thermostat-2.json) en la máquina local.

1. Inicie sesión en la aplicación IoT Central y vaya a la página **Plantillas de dispositivo**.

1. Seleccione **+ Nuevo**, **Dispositivo IoT** y **Siguiente: personalizar**.

1. Escriba *Termostato* como nombre de plantilla y seleccione **Siguiente: revisar**. Seleccione **Crear**.

1. Seleccione **Importar un modelo** e importe el archivo *thermostat-2.json* que descargó anteriormente.

1. A continuación, seleccione **Publicar** para publicar la plantilla del dispositivo.

Para crear una plantilla de dispositivo para un dispositivo de puerta de enlace IoT Edge:

1. Guarde una copia del manifiesto de implementación en la máquina de desarrollo local: [moduledeployment.json](https://raw.githubusercontent.com/iot-for-all/iot-central-transform-with-iot-edge/main/edgemodule/moduledeployment.json).

1. Abra la copia local del archivo de manifiesto *moduledeployment.json* en un editor de texto.

1. Busque la sección `registryCredentials` y reemplace los marcadores de posición por los valores que anotó al crear el registro de contenedor de Azure. El valor `address` es similar a `<username>.azurecr.io`.

1. Busque la sección `settings` para `transformmodule`. Reemplace `<acr or docker repo>` por el valor `address` que usó en el paso anterior. Guarde los cambios.

1. En la aplicación de IoT Central, vaya a la página **Plantillas de dispositivo**.

1. Seleccione **+ Nuevo**, **Azure IoT Edge** y, a continuación, seleccione **Siguiente: Personalizar**.

1. Escriba *dispositivo de puerta de enlace de IoT Edge* como el nombre de la plantilla de dispositivo. Seleccione **This is a gateway device** (Este es un dispositivo de puerta de enlace). Seleccione **Examinar** para cargar el archivo de manifiesto de implementación *moduledeployment.json* que editó anteriormente.

1. Cuando se valide el manifiesto de implementación, seleccione **Siguiente: Revisar** y, a continuación, **Crear**.

1. En **Modelo**, seleccione **Relaciones**. Seleccione **+ Agregar relación**. Escriba *Sensor de bajada* como nombre para mostrar y seleccione **Termostato** como destino. Seleccione **Guardar**.

1. A continuación, seleccione **Publicar** para publicar la plantilla del dispositivo.

Ahora tiene dos plantillas de dispositivo en la aplicación IoT Central. La plantilla **Dispositivo de puerta de enlace de IoT Edge** y la plantilla **Termostato** como dispositivo de bajada.

Para registrar un dispositivo de puerta de enlace en IoT Central:

1. En la aplicación IoT Central, vaya a la página **Dispositivos**.

1. Seleccione **Dispositivo de puerta de enlace IoT Edge** y **Crear un dispositivo**. Escriba *Dispositivo de puerta de enlace de IoT Edge* como el nombre del dispositivo, escriba *gateway-01* como id. del dispositivo y asegúrese de que el **dispositivo de puerta de enlace de IoT Edge** esté seleccionado como la plantilla de dispositivo. Seleccione **Crear**.

1. En la lista de dispositivos, haga clic en el **dispositivo de puerta de enlace de IoT Edge** y, a continuación, seleccione **Conectar**.

1. Tome nota de los valores del **ámbito de id.** , el **id. de dispositivo** y la **clave principal** para el **dispositivo de puerta de enlace de IoT Edge**. Los usará más adelante.

Para registrar un dispositivo de bajada en IoT Central:

1. En la aplicación IoT Central, vaya a la página **Dispositivos**.

1. Seleccione **Termostato** y **Crear un dispositivo**. Escriba *Termostato* como nombre del dispositivo, escriba *downstream-01* como el id. del dispositivo y asegúrese de que la opción **Termostato** esté seleccionada como la plantilla del dispositivo. Seleccione **Crear**.

1. En la lista de dispositivos, seleccione **Termostato** y, a continuación, **Attach to Gateway** (Adjuntar a la puerta de enlace). Seleccione la plantilla **Dispositivo de puerta de enlace de IoT Edge** y la instancia de **Dispositivo de puerta de enlace de IoT Edge**. Seleccione **Adjuntar**.

1. En la lista de dispositivos, haga clic en el **Termostato** y luego seleccione **Conectar**.

1. Tome nota de los valores del **ámbito de id.** , el **id. de dispositivo** y la **clave principal**  para el dispositivo **Termostato**. Los usará más adelante.

### <a name="deploy-the-gateway-and-downstream-devices"></a>Implementación de la puerta de enlace y los dispositivos

Para mayor comodidad, en este artículo se usan máquinas virtuales de Azure para ejecutar la puerta de enlace y los dispositivos de bajada. Para crear las dos máquinas virtuales de Azure, seleccione el botón **Implementar en Azure** a continuación, y use la información de la siguiente tabla para completar el formulario de **Implementación personalizada**:

| Campo | Value |
| ----- | ----- |
| Grupo de recursos | `ingress-scenario` |
| Puerta de enlace de prefijo de etiqueta DNS | Un nombre DNS único para esta máquina, como `<your name>edgegateway`. |
| Valor de bajada del prefijo de etiqueta DNS | Un nombre DNS único para esta máquina, como `<your name>downstream`. |
| El identificador de ámbito | El ámbito del identificador que anotó anteriormente. |
| Id. de dispositivo en la Puerta de enlace de IoT Edge | `gateway-01` |
| Clave de dispositivo en la puerta de enlace de IoT Edge | El valor de la clave principal que anotó anteriormente. |
| Tipo de autenticación | Contraseña |
| Contraseña o clave de administrador | Su elección de contraseña para la cuenta **AzureUser** en ambas máquinas virtuales. |

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure-Samples%2Fiot-central-docs-samples%2Fmaster%2Ftransparent-gateway%2FDeployGatewayVMs.json" target="_blank">
    <img src="https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.png" alt="Deploy to Azure button" />
</a>

Seleccione **Revisar y crear** y, a continuación, **Crear**. Se necesitan un par de minutos para crear las máquinas virtuales en el grupo de recursos del **escenario de entrada**.

Para comprobar que el dispositivo de IoT Edge esté funcionando correctamente:

1. Abra la aplicación IoT Central. A continuación, vaya al **dispositivo de puerta de enlace de IoT Edge** en la lista de dispositivos en la página **Dispositivos**.

1. Seleccione la pestaña **Módulos** y compruebe el estado de los tres módulos. El entorno de ejecución de IoT Edge tarda unos minutos en iniciarse en la máquina virtual. Cuando se inicia, el estado de los tres módulos es **En ejecución**. Si el entorno de ejecución de IoT Edge no se inicia, consulte [Solución de problemas del dispositivo de IoT Edge](../../iot-edge/troubleshoot.md).

Para que el dispositivo de IoT Edge funcione como puerta de enlace, necesita algunos certificados para demostrar su identidad en cualquier dispositivo de nivel inferior. En este artículo se utilizan certificados de demostración. En un entorno de producción, use certificados de su entidad de certificación.

Para generar los certificados de demostración e instalarlos en el dispositivo de puerta de enlace, haga lo siguiente:

1. Use SSH para conectarse e iniciar sesión en la máquina virtual del dispositivo de puerta de enlace. Puede encontrar el nombre DNS de esta máquina virtual en Azure Portal. Vaya a la máquina virtual **edgegateway** en el grupo de recursos del **escenario de entrada**.

    > [!TIP]
    > Es posible que deba abrir el puerto 22 para obtener acceso SSH en ambas máquinas virtuales antes de poder usar SSH para conectarse desde su máquina local o Azure Cloud Shell.

1. Ejecute los comandos siguientes para clonar el repositorio de IoT Edge y generar los certificados de demostración:

    ```bash
    # Clone the repo
    cd ~
    git clone https://github.com/Azure/iotedge.git

    # Generate the demo certificates
    mkdir certs
    cd certs
    cp ~/iotedge/tools/CACertificates/*.cnf .
    cp ~/iotedge/tools/CACertificates/certGen.sh .
    ./certGen.sh create_root_and_intermediate
    ./certGen.sh create_edge_device_ca_certificate "mycacert"
    ```

    Después de ejecutar los comandos anteriores, estos archivos están listos para usarse en los pasos siguientes:

    - *~/certs/certs/azure-iot-test-only.root.ca.cert.pem*: el certificado de CA raíz que se usa para crear todos los demás certificados de demostración con el fin de probar un escenario de IoT Edge.
    - *~/certs/certs/iot-edge-device-mycacert-full-chain.cert.pem*: un certificado de CA de dispositivo al que se hace referencia desde el archivo *config.yaml*. En un escenario de puerta de enlace, este certificado de CA es la forma en que el dispositivo IoT Edge comprueba su identidad en los dispositivos de nivel inferior.
    - *~/certs/private/iot-edge-device-mycacert.key.pem*: la clave privada asociada con el certificado de CA del dispositivo.

    Para más información sobre estos certificados de demostración, consulte [Creación de certificados de demostración para probar las características de dispositivo IoT Edge](../../iot-edge/how-to-create-test-certificates.md).

1. Abra el archivo *config.yaml* en un editor de texto. Por ejemplo:

    ```bash
    sudo nano /etc/iotedge/config.yaml
    ```

1. Ubique el valor `Certificate settings`. Quite la marca de comentario y modifique la configuración del certificado como se indica a continuación:

    ```text
    certificates:
      device_ca_cert: "file:///home/AzureUser/certs/certs/iot-edge-device-ca-mycacert-full-chain.cert.pem"
      device_ca_pk: "file:///home/AzureUser/certs/private/iot-edge-device-ca-mycacert.key.pem"
      trusted_ca_certs: "file:///home/AzureUser/certs/certs/azure-iot-test-only.root.ca.cert.pem"
    ```

    En el ejemplo anterior, se presupone que inició sesión como **AzureUser** y creó un certificado de CA de dispositivo denominado "mycacert".

1. Guarde los cambios y reinicie el entorno de ejecución de Azure IoT Edge:

    ```bash
    sudo systemctl restart iotedge
    ```

Si el entorno de ejecución de Azure IoT Edge se inicia correctamente después de los cambios, el estado de los módulos **$edgeAgent** y **$edgeHub** cambia a **En ejecución**. Puede ver estos valores de estado en la página **Módulos** del dispositivo de puerta de enlace en IoT Central.

Si el entorno de ejecución no se inicia, compruebe los cambios que realizó en *config.yaml* y consulte [Solución de problemas del dispositivo IoT Edge](../../iot-edge/troubleshoot.md).

### <a name="connect-downstream-device-to-iot-edge-device"></a>Conexión del dispositivo de bajada al dispositivo de IoT Edge

Para conectar un dispositivo de bajada al dispositivo de puerta de enlace de IoT Edge:

1. Use SSH para conectarse e iniciar sesión en la máquina virtual del dispositivo de bajada. Puede encontrar el nombre DNS de esta máquina virtual en Azure Portal. Vaya a la máquina virtual **leafdevice** en el grupo de recursos del **escenario de entrada**.

    > [!TIP]
    > Es posible que deba abrir el puerto 22 para obtener acceso SSH en ambas máquinas virtuales antes de poder usar SSH para conectarse desde su máquina local o Azure Cloud Shell.

1. Para clonar el repositorio de GitHub con el código fuente del dispositivo de bajada de ejemplo, ejecute el siguiente comando:

    ```bash
    cd ~
    git clone https://github.com/iot-for-all/iot-central-transform-with-iot-edge
    ```

1. Para copiar el certificado necesario desde el dispositivo de puerta de enlace, ejecute los siguientes `scp` comandos. Este comando `scp` usa el nombre de host `edgegateway` para identificar la máquina virtual de la puerta de enlace. Se le pedirá la contraseña:

    ```bash
    cd ~/iot-central-transform-with-iot-edge
    scp AzureUser@edgegateway:/home/AzureUser/certs/certs/azure-iot-test-only.root.ca.cert.pem leafdevice/certs
    ```

1. Vaya hasta la carpeta *leafdevice* e instale los paquetes necesarios. A continuación, ejecute los scripts `build` y `start` para aprovisionar y conectar el dispositivo a la puerta de enlace:

    ```bash
    cd ~/iot-central-transform-with-iot-edge/leafdevice
    sudo apt update
    sudo apt install nodejs npm node-typescript
    npm install
    npm run-script build
    npm run-script start
    ```

1. Escriba el id. de dispositivo, el id. de ámbito y la clave SAS del dispositivo de bajada que creó anteriormente. Para el nombre de host, escriba `edgegateway`. La salida de este comando tiene un aspecto similar al siguiente:

    ```output
    Registering device downstream-01 with scope 0ne00284FD9
    Registered device downstream-01.
    Connecting device downstream-01
    Connected device downstream-01
    Sent telemetry for device downstream-01
    Sent telemetry for device downstream-01
    Sent telemetry for device downstream-01
    Sent telemetry for device downstream-01
    Sent telemetry for device downstream-01
    Sent telemetry for device downstream-01
    ```

### <a name="verify"></a>Verify

Para comprobar que el escenario se esté ejecutando, vaya hasta su **dispositivo de puerta de enlace de IoT Edge** en IoT Central:

:::image type="content" source="media/howto-transform-data/transformed-data.png" alt-text="Captura de pantalla que muestra los datos transformados en la página de dispositivos.":::

- Seleccione **Módulos**. Compruebe que los tres módulos de IoT Edge **$edgeAgent**, **$edgeHub** y **transformmodule** se estén ejecutando.
- Seleccione **Dispositivos de bajada** y compruebe que el dispositivo de bajada está aprovisionado.
- Seleccione **Datos sin procesar**. Los datos de telemetría de la columna **Datos sin modelo** tienen el siguiente aspecto:

    ```json
    {"device":{"deviceId":"downstream-01"},"measurements":{"temperature":85.21208,"pressure":59.97321,"humidity":77.718124,"scale":"farenheit"}}
    ```

Dado que el dispositivo IoT Edge está transformando los datos del dispositivo de bajada, la telemetría está asociada al dispositivo de puerta de enlace en IoT Central. Para visualizar la telemetría, cree una nueva versión de la plantilla del **dispositivo de puerta de enlace IoT Edge** mediante definiciones para los tipos de telemetría.

## <a name="data-transformation-at-egress"></a>Transformación de datos en la salida

Puede conectar los dispositivos a IoT Central, exportar los datos del dispositivo a un motor de proceso para transformarlos y luego enviar los datos transformados a IoT Central para poder realizar la administración y el análisis del dispositivo. Por ejemplo:

- Los dispositivos envían datos de ubicación a IoT Central.
- IoT Central exporta esos datos a un motor de proceso que mejora los datos de ubicación con información meteorológica.
- El motor de proceso devuelve los datos mejorados a IoT Central.

Puede usar el [puente de dispositivos de IoT Central](https://github.com/Azure/iotc-device-bridge) como motor de proceso para transformar los datos exportados desde IoT Central.

Una ventaja de transformar los datos en la salida es que los dispositivos se conectan directamente a IoT Central, lo que facilita el envío de comandos a los dispositivos o la actualización de las propiedades del dispositivo. Sin embargo, con este método, es posible que use más mensajes que la asignación mensual y que por ello aumente el costo de usar Azure IoT Central.

### <a name="use-the-iot-central-device-bridge-to-transform-device-data"></a>Uso del puente de dispositivos de IoT Central para transformar los datos del dispositivo

:::image type="content" source="media/howto-transform-data/transform-egress.png" alt-text="Transformación de datos en la salida mediante IoT Edge" border="false":::

En este escenario, un motor de proceso transforma los datos del dispositivo exportados de IoT Central antes de enviarlos de vuelta a la aplicación de IoT Central. Estos son los pasos de alto nivel que se deben realizar para configurar este escenario:

1. **Configuración del motor de proceso:** cree un puente de dispositivos de IoT Central para que actúe como motor de proceso para la transformación de datos.

1. **Transformación de los datos del dispositivo en el puente de dispositivos:** transforme los datos en el puente de dispositivos; para ello, debe modificar el código de función del puente de dispositivos para el caso de uso de transformación de datos.

1. **Habilitación del flujo de datos de IoT Central al puente de dispositivos:** exporte los datos de IoT Central al puente de dispositivos para su transformación. A continuación, reenvíe los datos transformados a IoT Central. Al crear la exportación de datos, use filtros de propiedades de mensaje para exportar solo los datos que no se hayan exportado.

1. **Comprobación**: conecte su dispositivo a la aplicación IoT Central y compruebe tanto los datos sin procesar del dispositivo como los datos transformados en IoT Central.

<!-- To Do - doesn't the device send JSON data? -->
En el ejemplo descrito en las secciones siguientes, el dispositivo de bajada envía datos CSV con el formato siguiente al dispositivo de puerta de enlace de IoT Edge:

```csv
"<temperature in degrees C>, <humidity>, <latitude>, <longitude>"
```

Use el puente de dispositivos para transformar los datos del dispositivo mediante:

- El cambio de la unidad de temperatura de centígrados a Fahrenheit.
- Realizando el enriquecimiento de los datos del dispositivo con los datos meteorológicos que extrajo del servicio [Open Weather](https://openweathermap.org/) para obtener los valores de latitud y longitud.

A continuación, el puente de dispositivos envía los datos transformados a IoT Central con el siguiente formato:

```json
{
  "temp": <temperature in degrees F>,
  "humidity": <humidity>,
  "lat": <latitude>,
  "lon": <logitude>,
  "weather": {
    "weather_temp": <temperature at lat/lon>,
    "weather_humidity": <humidity at lat/lon>,
    "weather_pressure": <pressure at lat/lon>,
    "weather_windspeed": <wind speed at lat/lon>,
    "weather_clouds": <cloud cover at lat/lon>,
    "weather_uvi": <UVI at lat/lon>
  }
}
```

En los pasos siguientes se muestra cómo instalar y configurar este escenario:

### <a name="retrieve-your-iot-central-connection-settings"></a>Recuperación de la configuración de conexión de IoT Central

Antes de configurar este escenario, debe obtener algunas configuraciones de conexión de la aplicación de IoT Central:

1. Inicie sesión en su aplicación de IoT Central.

1. Vaya a **Administración> Conexión del dispositivo**.

1. Anote el **ámbito del id.** . Necesitará este valor más adelante.

1. Seleccione el grupo de inscripción **SaS-IoT-Devices**. Anote la clave principal de la firma de acceso compartido. Necesitará este valor más adelante.

### <a name="set-up-a-compute-engine"></a>Configuración de un motor de proceso

En este escenario se usa la misma implementación de Azure Functions que el puente de dispositivos de IoT Central. Para implementar el puente de dispositivos, seleccione el botón **Implementar en Azure** y, a continuación, use la información de la siguiente tabla para completar el formulario de **Implementación personalizada**:

| Campo | Value |
| ----- | ----- |
| Grupo de recursos | Cree un nuevo grupo de recursos denominado `egress-scenario`. |
| Region | Seleccione la región más cercana a la suya. |
| El identificador de ámbito | Use el **ámbito del id.** que anotó anteriormente. |
| Clave SAS de IoT Central | Use la clave principal de la firma de acceso compartido para el grupo de inscripción **SaS-IoT-Devices**. Anotó este valor anteriormente. |

[![Impleméntelo en Azure](http://azuredeploy.net/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fiotc-device-bridge%2Fmaster%2Fazuredeploy.json).

Seleccione **Revisar y crear** y, a continuación, **Crear**. Se necesitan un par de minutos para crear la función de Azure y los recursos relacionados en el grupo de recursos **escenario de salida**.

### <a name="transform-device-data-in-the-device-bridge"></a>Transformación de los datos del dispositivo en el puente de dispositivos

Para configurar el puente de dispositivos para transformar los datos exportados del dispositivo:

1. Obtenga una clave de API de aplicación del servicio Open Weather. La cuenta es gratuita, pero tiene un uso limitado del servicio. Para crear una clave de API de aplicación, cree una cuenta en el [Portal del servicio Open Weather](https://openweathermap.org/) y siga las instrucciones. Deberá usar la clave de API de Open Weather más adelante.

1. En Azure Portal, vaya hasta la aplicación de funciones en el grupo de recursos del **escenario de salida**.

1. En el menú de navegación de la izquierda, en **Herramientas de desarrollo**, seleccione **Editor de App Service (versión preliminar)** .

1. Seleccione **Ir&rarr;** para abrir la página del **Editor de App Service**. Se han realizado los siguientes cambios:

    1. Abra el archivo *wwwroot/IoTCIntegration/index.js*. Reemplace todo el código de este archivo con el código de [index.js](https://raw.githubusercontent.com/iot-for-all/iot-central-compute/main/Azure_function/index.js).

    1. En el nuevo archivo *index.js*, actualice el archivo de variable `openWeatherAppId` con la clave de API de Open Weather que obtuvo anteriormente.

        ```javascript
        const openWeatherAppId = '<Your Open Weather API Key>'
        ```

    1. Agregue una propiedad de mensaje a los datos que envió la función a IoT Central. IoT Central usa esta propiedad para evitar la exportación de los datos transformados. Para realizar este cambio, abra el archivo *wwwroot/IoTCIntegration/lib/engine.js*. Busque el código siguiente:

        ```javascript
        if (timestamp) {
          message.properties.add('iothub-creation-time-utc', timestamp);
        }
        ```

        Agregue el siguiente código justo después del código del fragmento anterior:

        ```javascript
        // add a message property that we can look for in data export to not re-compute computed telemetry
        message.properties.add('computed', true);
        ```

        Como referencia, puede ver un ejemplo completo del archivo [engine.js](https://raw.githubusercontent.com/iot-for-all/iot-central-compute/main/Azure_function/lib/engine.js).

1. En el **Editor de App Service**, seleccione **Consola** en el panel de navegación de la izquierda. Ejecute los comandos siguientes para instalar los paquetes necesarios:

    ```bash
    cd IoTCIntegration
    npm install
    ```

    Este comando puede tardar varios minutos en ejecutarse.

1. Vuelva a la página **Información general de la función de Azure** y reinicie la función:

    :::image type="content" source="media/howto-transform-data/azure-function.png" alt-text="Reinicio de la función":::

1. Seleccione **Functions** en el menú de navegación de la izquierda. A continuación, seleccione **IoTCIntegration**. Seleccione **Código y prueba**.

1. Anote la dirección URL de la función, ya que necesitará este valor más adelante:

    :::image type="content" source="media/howto-transform-data/get-function-url.png" alt-text="Obtención de la dirección URL de la función":::

### <a name="enable-data-flow-from-iot-central-to-the-device-bridge"></a>Habilitación del flujo de datos de IoT Central al puente de dispositivos

En esta sección se describe cómo configurar la aplicación Azure IoT Central.

En primer lugar, guarde el archivo del [modelo de dispositivo](https://raw.githubusercontent.com/iot-for-all/iot-central-compute/main/model.json) en el equipo local.

Para agregar una plantilla de dispositivo a la aplicación de IoT Central, vaya hasta la aplicación de IoT Central y a continuación:

1. Inicie sesión en la aplicación IoT Central y vaya a la página **Plantillas de dispositivo**.

1. Seleccione **+ Nuevo**, **Dispositivo de IoT**, **Siguiente: personalizar** y escriba *Modelo de proceso*  como nombre de la plantilla. Seleccione **Siguiente: Review** (Siguiente: revisar). Seleccione **Crear**.

1. Seleccione **Importar un modelo** y busque el archivo *model.json* que descargó anteriormente.

1. Una vez importado el modelo, seleccione **Publicar** para publicar la plantilla de dispositivo **Modelo de proceso**.

Para configurar la exportación de datos para enviarlos al puente de dispositivos:

1. En la aplicación de IoT Central, seleccione **Exportación de datos**.

1. Seleccione **+ Nuevo destino** para crear un destino que se usará con el puente de dispositivos. Llame al destino *Función de proceso*, y en **Tipo de destino** seleccione **Webhook**. Para la dirección URL de devolución de llamada, seleccione Pegar en la URL de función que anotó anteriormente. Deje la opción **Autorización**  como **Sin autorización**.

1. Guarde los cambios.

1. Seleccione **+ Nueva exportación** y cree una exportación de datos denominada *Exportación de procesos*.

1. Agregue un filtro para exportar solo los datos del dispositivo para la plantilla de dispositivos que esté usando. Seleccione **+ Filtro**, el elemento **Plantilla de dispositivo**, el operador **Igual a** y la plantilla de dispositivo **Modelo de proceso** que acaba de crear.

1. Agregue un filtro de mensaje para diferenciar entre los datos transformados y los no transformados. Este filtro impide que los valores transformados se envíen de nuevo al puente del dispositivo. Seleccione **+ Filtro de propiedad del mensaje** y escriba el valor del nombre *procesado*; a continuación, seleccione el operador **No existe**. La cadena `computed` se usa como palabra clave en el código de ejemplo del puente de dispositivos.

1. Para el destino, seleccione el destino de la **Función de proceso** que creó anteriormente.

1. Guarde los cambios. Después de aproximadamente un minuto, el **Estado de exportación** se muestra como **Correcto**.

### <a name="verify"></a>Verify

El dispositivo de ejemplo que se usa para probar el escenario se escribe en Node.js. Asegúrese de que tiene Node.js y NPM instalados en su máquina local. Si no quiere instalar estos requisitos previos, use [Azure Cloud Shell](https://shell.azure.com/), ya que los tiene preinstalados.

Para ejecutar un dispositivo de ejemplo que pruebe el escenario:

1. Clone el repositorio de GitHub que contiene el código fuente del módulo y ejecute el siguiente comando:

    ```bash
    git clone https://github.com/iot-for-all/iot-central-compute
    ```

1. Para conectar el dispositivo de ejemplo a la aplicación IoT Central, edite la configuración de conexión en el archivo *iot-central-compute/device/device.js*. Reemplace el id. de ámbito y la clave SAS de grupo con los valores que anotó anteriormente:

    ```javascript
    // These values need to be filled in from your Azure IoT Central application
    //
    const scopeId = "<IoT Central Scope Id value>";
    const groupSasKey = "<IoT Central Group SAS key>";
    //
    ```

    Guarde los cambios.

1. Use los siguientes comandos para instalar los paquetes necesarios y ejecutar el dispositivo:

    ```bash
    cd ~/iot-central-compute/device
    npm install
    node device.js
    ```

1. El resultado de este comando es similar al siguiente resultado:

    ```output
    registration succeeded
    assigned hub=iotc-2bd611b0....azure-devices.net
    deviceId=computeDevice
    Client connected
    send status: MessageEnqueued [{"data":"33.23, 69.09, 30.7213, -61.1192"}]
    send status: MessageEnqueued [{"data":"2.43, 75.86, -2.6358, 162.935"}]
    send status: MessageEnqueued [{"data":"6.19, 76.55, -14.3538, -82.314"}]
    send status: MessageEnqueued [{"data":"33.26, 48.01, 71.9172, 48.6606"}]
    send status: MessageEnqueued [{"data":"40.5, 36.41, 14.6043, 14.079"}]
    ```

1. En la aplicación IoT Central, vaya al dispositivo denominado **computeDevice**. En la vista **Datos sin procesar**, tiene dos flujos de telemetría diferentes que aparecen cada cinco segundos. La secuencia con datos sin modelar es la telemetría original, y la secuencia con datos modelados son los datos que transformó la función:

    :::image type="content" source="media/howto-transform-data/egress-telemetry.png" alt-text="Captura de pantalla que muestra los datos sin procesar originales y transformados.":::

## <a name="clean-up-resources"></a>Limpieza de recursos

Si ya no necesita los recursos de Azure que creó al seguir los pasos de esta guía, elimine los [grupos de recursos de Azure Portal](https://portal.azure.com/?r=1#blade/HubsExtension/BrowseResourceGroups).

Los dos grupos de recursos que usó en esta guía son el **escenario de entrada** y el **escenario de salida**.

## <a name="next-steps"></a>Pasos siguientes

En este artículo obtuvo información sobre las diferentes opciones para transformar los datos del dispositivo para IoT Central, tanto en la entrada como en la salida. En el artículo se incluyen tutoriales para dos escenarios específicos:

- Use un módulo de IoT Edge para transformar los datos de los dispositivos de bajada antes de enviar los datos a la aplicación IoT Central.
- Use Azure Functions para transformar los datos fuera de IoT Central. En este escenario, IoT Central usa una exportación de datos para enviar datos entrantes a una función de Azure que se va a transformar. La función devuelve los datos transformados a la aplicación IoT Central.

Ahora que ha aprendido a transformar los datos del dispositivo fuera de la aplicación Azure IoT Central, puede aprender a usar el [análisis para analizar los datos del dispositivo en IoT Central](howto-create-analytics.md).
