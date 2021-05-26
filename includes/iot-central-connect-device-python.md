---
author: dominicbetts
ms.author: dobett
ms.service: iot-pnp
ms.topic: include
ms.date: 03/31/2021
ms.openlocfilehash: cee499085bf4196f3cd49b4307abd34c130c8f5a
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110487561"
---
[![Examinar el código](../articles/iot-central/core/media/common/browse-code.svg)](https://github.com/Azure/azure-iot-sdk-python/tree/master/azure-iot-device/samples/pnp)

## <a name="prerequisites"></a>Requisitos previos

Para completar los pasos de este artículo, necesitará los siguientes recursos:

[!INCLUDE [iot-central-prerequisites-basic](iot-central-prerequisites-basic.md)]

- Una máquina de desarrollo que tenga instalada la versión 3.7 de [Python](https://www.python.org/) o una posterior. Puede ejecutar `python --version` en la línea de comandos para comprobar la versión. Python está disponible para una amplia variedad de sistemas operativos. En las instrucciones de este tutorial se da por hecho que se ejecuta el comando **python** en el símbolo del sistema de Windows.

- Una copia local del repositorio de GitHub de [SDK de IoT de Microsoft Azure para Python](https://github.com/Azure/azure-iot-sdk-python) que contiene el código de ejemplo. Use este vínculo para descargar una copia del repositorio: [Descargar el archivo ZIP](https://github.com/Azure/azure-iot-sdk-python/archive/master.zip). Después, descomprima el archivo en una ubicación conveniente en la máquina local.

## <a name="review-the-code"></a>Revisión del código

En la copia del SDK de Microsoft Azure IoT para Python que descargó anteriormente, abra el archivo *azure-iot-sdk-python/azure-iot-device/samples/pnp/temp_controller_with_thermostats.py* en un editor de texto.

Al ejecutar el ejemplo para conectarse a IoT Central, usa Device Provisioning Service (DPS) para registrar el dispositivo y generar una cadena de conexión. En el ejemplo se recupera la información de conexión de DPS que necesita del entorno de línea de comandos.

La función `main`:

* Usa DPS para aprovisionar el dispositivo. La información de aprovisionamiento incluye el identificador de modelo. IoT Central usa el identificador de modelo para identificar o generar la plantilla de dispositivo específico. Para más información, consulte [¿Cómo se asocia un dispositivo a una plantilla de dispositivo?](../articles/iot-central/core/concepts-get-connected.md#associate-a-device-with-a-device-template)
* Crea un objeto `Device_client` y establece el identificador del modelo `dtmi:com:example:TemperatureController;2` antes de abrir la conexión.
* Envía los valores de propiedad iniciales a IoT Central. Utiliza `pnp_helper` para crear las revisiones.
* Crea agentes de escucha para los comandos `getMaxMinReport` y `reboot`. Cada componente de termostato tiene su propio comando `getMaxMinReport`.
* Crea un agente de escucha de propiedad para realizar escuchas de las actualizaciones de las propiedades de escritura.
* Inicia un bucle para enviar telemetría de temperatura desde los dos componentes del termostato y la telemetría del conjunto de trabajo desde el componente predeterminado cada 8 segundos.

```python
async def main():
    switch = os.getenv("IOTHUB_DEVICE_SECURITY_TYPE")
    if switch == "DPS":
        provisioning_host = (
            os.getenv("IOTHUB_DEVICE_DPS_ENDPOINT")
            if os.getenv("IOTHUB_DEVICE_DPS_ENDPOINT")
            else "global.azure-devices-provisioning.net"
        )
        id_scope = os.getenv("IOTHUB_DEVICE_DPS_ID_SCOPE")
        registration_id = os.getenv("IOTHUB_DEVICE_DPS_DEVICE_ID")
        symmetric_key = os.getenv("IOTHUB_DEVICE_DPS_DEVICE_KEY")

        registration_result = await provision_device(
            provisioning_host, id_scope, registration_id, symmetric_key, model_id
        )

        if registration_result.status == "assigned":
            print("Device was assigned")
            print(registration_result.registration_state.assigned_hub)
            print(registration_result.registration_state.device_id)
            device_client = IoTHubDeviceClient.create_from_symmetric_key(
                symmetric_key=symmetric_key,
                hostname=registration_result.registration_state.assigned_hub,
                device_id=registration_result.registration_state.device_id,
                product_info=model_id,
            )
        else:
            raise RuntimeError(
                "Could not provision device. Aborting Plug and Play device connection."
            )

    elif switch == "connectionString":
        # ...

    # Connect the client.
    await device_client.connect()

    ################################################
    # Update readable properties from various components

    properties_root = pnp_helper.create_reported_properties(serialNumber=serial_number)
    properties_thermostat1 = pnp_helper.create_reported_properties(
        thermostat_1_component_name, maxTempSinceLastReboot=98.34
    )
    properties_thermostat2 = pnp_helper.create_reported_properties(
        thermostat_2_component_name, maxTempSinceLastReboot=48.92
    )
    properties_device_info = pnp_helper.create_reported_properties(
        device_information_component_name,
        swVersion="5.5",
        manufacturer="Contoso Device Corporation",
        model="Contoso 4762B-turbo",
        osName="Mac Os",
        processorArchitecture="x86-64",
        processorManufacturer="Intel",
        totalStorage=1024,
        totalMemory=32,
    )

    property_updates = asyncio.gather(
        device_client.patch_twin_reported_properties(properties_root),
        device_client.patch_twin_reported_properties(properties_thermostat1),
        device_client.patch_twin_reported_properties(properties_thermostat2),
        device_client.patch_twin_reported_properties(properties_device_info),
    )

    ################################################
    # Get all the listeners running
    print("Listening for command requests and property updates")

    global THERMOSTAT_1
    global THERMOSTAT_2
    THERMOSTAT_1 = Thermostat(thermostat_1_component_name, 10)
    THERMOSTAT_2 = Thermostat(thermostat_2_component_name, 10)

    listeners = asyncio.gather(
        execute_command_listener(
            device_client, method_name="reboot", user_command_handler=reboot_handler
        ),
        execute_command_listener(
            device_client,
            thermostat_1_component_name,
            method_name="getMaxMinReport",
            user_command_handler=max_min_handler,
            create_user_response_handler=create_max_min_report_response,
        ),
        execute_command_listener(
            device_client,
            thermostat_2_component_name,
            method_name="getMaxMinReport",
            user_command_handler=max_min_handler,
            create_user_response_handler=create_max_min_report_response,
        ),
        execute_property_listener(device_client),
    )

    ################################################
    # Function to send telemetry every 8 seconds

    async def send_telemetry():
        print("Sending telemetry from various components")

        while True:
            curr_temp_ext = random.randrange(10, 50)
            THERMOSTAT_1.record(curr_temp_ext)

            temperature_msg1 = {"temperature": curr_temp_ext}
            await send_telemetry_from_temp_controller(
                device_client, temperature_msg1, thermostat_1_component_name
            )

            curr_temp_int = random.randrange(10, 50)  # Current temperature in Celsius
            THERMOSTAT_2.record(curr_temp_int)

            temperature_msg2 = {"temperature": curr_temp_int}

            await send_telemetry_from_temp_controller(
                device_client, temperature_msg2, thermostat_2_component_name
            )

            workingset_msg3 = {"workingSet": random.randrange(1, 100)}
            await send_telemetry_from_temp_controller(device_client, workingset_msg3)

    send_telemetry_task = asyncio.ensure_future(send_telemetry())

    # ...
```

La función `provision_device` usa DPS para aprovisionar el dispositivo y registrarlo con IoT Central. La función incluye el identificador del modelo de dispositivo, que IoT Central usa para [asociar un dispositivo a una plantilla de dispositivo](../articles/iot-central/core/concepts-get-connected.md#associate-a-device-with-a-device-template) en la carga útil de aprovisionamiento:

```python
async def provision_device(provisioning_host, id_scope, registration_id, symmetric_key, model_id):
    provisioning_device_client = ProvisioningDeviceClient.create_from_symmetric_key(
        provisioning_host=provisioning_host,
        registration_id=registration_id,
        id_scope=id_scope,
        symmetric_key=symmetric_key,
    )

    provisioning_device_client.provisioning_payload = {"modelId": model_id}
    return await provisioning_device_client.register()
```

La función `execute_command_listener` controla las solicitudes de comando, ejecuta la función `max_min_handler` cuando el dispositivo recibe el comando `getMaxMinReport` para los componentes del termostato y la función `reboot_handler` cuando el dispositivo recibe el comando `reboot`. Usa el módulo `pnp_helper` para generar la respuesta:

```python
async def execute_command_listener(
    device_client,
    component_name=None,
    method_name=None,
    user_command_handler=None,
    create_user_response_handler=None,
):
    while True:
        if component_name and method_name:
            command_name = component_name + "*" + method_name
        elif method_name:
            command_name = method_name
        else:
            command_name = None

        command_request = await device_client.receive_method_request(command_name)
        print("Command request received with payload")
        values = command_request.payload
        print(values)

        if user_command_handler:
            await user_command_handler(values)
        else:
            print("No handler provided to execute")

        (response_status, response_payload) = pnp_helper.create_response_payload_with_status(
            command_request, method_name, create_user_response=create_user_response_handler
        )

        command_response = MethodResponse.create_from_method_request(
            command_request, response_status, response_payload
        )

        try:
            await device_client.send_method_response(command_response)
        except Exception:
            print("responding to the {command} command failed".format(command=method_name))
```

`async def execute_property_listener` controla las actualizaciones de las propiedades de escritura como `targetTemperature` para los componentes del termostato y genera la respuesta JSON. Usa el módulo `pnp_helper` para generar la respuesta:

```python
async def execute_property_listener(device_client):
    while True:
        patch = await device_client.receive_twin_desired_properties_patch()  # blocking call
        print(patch)
        properties_dict = pnp_helper.create_reported_properties_from_desired(patch)

        await device_client.patch_twin_reported_properties(properties_dict)
```

La función `send_telemetry_from_temp_controller` envía los mensajes de telemetría de los componentes del termostato a IoT Central. Usa el módulo `pnp_helper` para crear los mensajes:

```python
async def send_telemetry_from_temp_controller(device_client, telemetry_msg, component_name=None):
    msg = pnp_helper.create_telemetry(telemetry_msg, component_name)
    await device_client.send_message(msg)
    print("Sent message")
    print(msg)
    await asyncio.sleep(5)
```

## <a name="get-connection-information"></a>Obtención de información sobre la conexión

[!INCLUDE [iot-central-connection-configuration](iot-central-connection-configuration.md)]

## <a name="run-the-code"></a>Ejecución del código

Para ejecutar la aplicación de ejemplo, abra un entorno de línea de comandos y vaya a la carpeta *azure-iot-sdk-python/azure-iot-device/samples/pnp* que contiene el archivo de ejemplo *temp_controller_with_thermostats.py*.

[!INCLUDE [iot-central-connection-environment](iot-central-connection-environment.md)]

Instale los paquetes necesarios:

```cmd/sh
pip install azure-iot-device
```

Ejecución del ejemplo:

```cmd/sh
python temp_controller_with_thermostats.py
```

La salida siguiente muestra el registro del dispositivo y la conexión a IoT Central. El ejemplo envía las propiedades `maxTempSinceLastReboot` de los dos componentes del termostato antes de comenzar a enviar telemetría:

```cmd/sh
Device was assigned
iotc-60a.....azure-devices.net
sample-device-01
Updating pnp properties for root interface
{'serialNumber': 'alohomora'}
Updating pnp properties for thermostat1
{'thermostat1': {'maxTempSinceLastReboot': 98.34, '__t': 'c'}}
Updating pnp properties for thermostat2
{'thermostat2': {'maxTempSinceLastReboot': 48.92, '__t': 'c'}}
Updating pnp properties for deviceInformation
{'deviceInformation': {'swVersion': '5.5', 'manufacturer': 'Contoso Device Corporation', 'model': 'Contoso 4762B-turbo', 'osName': 'Mac Os', 'processorArchitecture': 'x86-64', 'processorManufacturer': 'Intel', 'totalStorage': 1024, 'totalMemory': 32, '__t': 'c'}}
Listening for command requests and property updates
Press Q to quit
Sending telemetry from various components
Sent message
{"temperature": 27}
Sent message
{"temperature": 17}
Sent message
{"workingSet": 13}
```

[!INCLUDE [iot-central-monitor-thermostat](iot-central-monitor-thermostat.md)]

Puede ver cómo responde el dispositivo a los comandos y las actualizaciones de propiedades:

```cmd/sh
{'thermostat1': {'targetTemperature': 67, '__t': 'c'}, '$version': 2}
the data in the desired properties patch was: {'thermostat1': {'targetTemperature': 67, '__t': 'c'}, '$version': 2}
Values received are :-
{'targetTemperature': 67, '__t': 'c'}
Sent message

...

Command request received with payload
2021-03-31T05:00:00.000Z
Will return the max, min and average temperature from the specified time 2021-03-31T05:00:00.000Z to the current time
Done generating
{"avgTemp": 4.0, "endTime": "2021-03-31T12:29:48.322427", "maxTemp": 18, "minTemp": null, "startTime": "2021-03-31T12:28:28.322381"}
```
