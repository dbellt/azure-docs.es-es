---
title: Conexión de una puerta de enlace IoT Edge transparente a una aplicación de Azure IoT Central
description: Cómo conectar dispositivos a través de una puerta de enlace IoT Edge transparente a una aplicación de IoT Central
author: dominicbetts
ms.author: dobett
ms.date: 03/08/2021
ms.topic: how-to
ms.service: iot-central
services: iot-central
ms.custom: device-developer
ms.openlocfilehash: 4e88ad58c7baba1c66c30df3f4effdbf11371c18
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105045338"
---
# <a name="how-to-connect-devices-through-an-iot-edge-transparent-gateway"></a>Cómo conectar dispositivos a través de una puerta de enlace IoT Edge transparente

Un dispositivo IoT Edge puede servir como una puerta de enlace que proporciona una conexión entre otros dispositivos de una red local y la aplicación de IoT Central. Cuando el dispositivo no puede acceder directamente a la aplicación de IoT Central, se usa una puerta de enlace.

IoT Edge admite los patrones de puerta de enlace [*transparente* y de *traducción*](../../iot-edge/iot-edge-as-gateway.md). En este artículo, se resume cómo implementar el patrón de puerta de enlace transparente. En este patrón, la puerta de enlace pasa los mensajes desde el dispositivo de nivel inferior hasta el punto de conexión de IoT Hub en la aplicación de IoT Central.

En este artículo, se usan máquinas virtuales para hospedar la puerta de enlace y el dispositivo de nivel inferior. En un escenario real, la puerta de enlace y el dispositivo de nivel inferior se ejecutarían en dispositivos físicos de la red local.

## <a name="prerequisites"></a>Requisitos previos

Para completar los pasos de este tutorial, deberá tener una suscripción de Azure activa.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

Complete el inicio rápido [Creación de una aplicación de Azure IoT Central](./quick-deploy-iot-central.md) para crear una aplicación de IoT Central mediante la plantilla **Aplicación personalizada > Aplicación personalizada**.

Para seguir los pasos de este artículo, descargue los archivos siguientes al equipo:

- [Modelo de dispositivo de termostato](https://raw.githubusercontent.com/Azure/iot-plugandplay-models/main/dtmi/com/example/thermostat-1.json)
- [Manifiesto de puerta de enlace transparente](https://raw.githubusercontent.com/Azure-Samples/iot-central-docs-samples/master/transparent-gateway/EdgeTransparentGatewayManifest.json)

## <a name="add-device-templates"></a>Incorporación de plantillas de dispositivo

Los dispositivos de nivel inferior y el dispositivo de puerta de enlace requieren plantillas de dispositivo en IoT Central. IoT Central permite modelar la relación entre los dispositivos de nivel inferior y la puerta de enlace para que pueda verlos y administrarlos una vez conectados.

Si desea crear una plantilla de dispositivo para un dispositivo de nivel inferior, cree una plantilla de dispositivo estándar que modele las funcionalidades del dispositivo. En el ejemplo que se muestra en este artículo se utiliza el modelo de dispositivo de termostato.

Para crear una plantilla de dispositivo para un dispositivo de nivel inferior, siga estos pasos:

1. Cree una plantilla de dispositivo y elija **Dispositivo IoT** como el tipo de plantilla.

1. En la página **Personalizar** del Asistente, escriba un nombre como *Termostato* para la plantilla de dispositivo.

1. Después de crear la plantilla de dispositivo, seleccione **Importar un modelo**. Seleccione un modelo como el archivo *thermostat-1.json* que descargó anteriormente.

1. Para generar algunas vistas predeterminadas para el termostato, seleccione las vistas y, luego, elija **Generar vistas predeterminadas**.

1. Publicar la plantilla de dispositivo.

Para crear una plantilla de dispositivo para una puerta de enlace IoT Edge transparente:

1. Cree una plantilla de dispositivo y elija **Azure IoT Edge** como el tipo de plantilla.

1. En la página **Personalizar** del Asistente, escriba un nombre como *Puerta de enlace Edge* para la plantilla de dispositivo.

1. En la página **Personalizar** del Asistente, active **Dispositivo de puerta de enlace con dispositivos de nivel inferior**.

1. En la página **Personalizar** del Asistente, seleccione **Examinar**. Cargue el archivo *EdgeTransparentGatewayManifest.json* que descargó anteriormente.

1. Agregue una entrada en **Relaciones** a la plantilla de dispositivo de nivel inferior.

En la captura de pantalla siguiente se muestra la página **Relaciones** de un dispositivo de puerta de enlace IoT Edge que tiene dispositivos de nivel inferior que usan la plantilla de dispositivo **Termostato**:

:::image type="content" source="media/how-to-connect-iot-edge-transparent-gateway/device-template-relationship.png" alt-text="Captura de pantalla que muestra la relación de la plantilla de dispositivo de puerta de enlace IoT Edge con una plantilla de dispositivo de nivel inferior de termostato.":::

En la captura de pantalla anterior se muestra una plantilla de dispositivo de puerta de enlace IoT Edge sin módulos definidos. Una puerta de enlace transparente no requiere ningún módulo, porque el entorno de ejecución de Azure IoT Edge reenvía los mensajes desde los dispositivos de nivel inferior a IoT Central. Si la puerta de enlace misma necesita enviar telemetría, sincronizar propiedades o controlar comandos, es posible definir estas funcionalidades en el componente predeterminado o en un módulo.

Agregue las vistas y propiedades de la nube que sean necesarias antes de publicar las plantillas de dispositivo de puerta de enlace y de nivel inferior.

## <a name="add-the-devices"></a>Incorporación de los dispositivos

Al agregar los dispositivos a la aplicación de IoT Central, puede definir la relación entre los dispositivos de nivel inferior y la puerta de enlace transparente.

Para agregar los dispositivos, siga estos pasos:

1. Vaya a la página de los dispositivos de la aplicación de IoT Central.

1. Agregue una instancia del dispositivo IoT Edge de puerta de enlace transparente. En este artículo, el id. de dispositivo de puerta de enlace es `edgegateway`.

1. Agregue una o más instancias del dispositivo de nivel inferior. En este artículo, los dispositivos de nivel inferior son termostatos con los id. `thermostat1` y `thermostat2`.

1. En la lista de dispositivos, seleccione cada dispositivo de nivel inferior y seleccione **Attach to gateway** (Conectar a la puerta de enlace).

En la captura de pantalla siguiente se muestra la lista de dispositivos conectados a una puerta de enlace en la página **Dispositivos de nivel inferior**:

:::image type="content" source="media/how-to-connect-iot-edge-transparent-gateway/downstream-devices.png" alt-text="Captura de pantalla que muestra la lista de dispositivos de nivel inferior conectados a una puerta de enlace transparente.":::

En una puerta de enlace transparente, los dispositivos de nivel inferior se conectan a la puerta de enlace misma y no a un módulo personalizado hospedado por la puerta de enlace.

Antes de implementar los dispositivos, necesita lo siguiente:

- El **ámbito de identificador** de la aplicación de IoT Central.
- Los valores de **id. de dispositivo** para los dispositivos de nivel inferior y de puerta de enlace.
- Los valores de **clave principal** para los dispositivos de nivel inferior y de puerta de enlace.

Para buscar estos valores, vaya a cada dispositivo en la lista de dispositivos y seleccione **Conectar**. Anote estos valores antes de continuar.

## <a name="deploy-the-gateway-and-devices"></a>Implementación de la puerta de enlace y los dispositivos

Para que pueda probar este escenario, en los pasos siguientes se muestra cómo implementar los dispositivos de nivel inferior y la puerta de enlace en máquinas virtuales de Azure. En un escenario real, la puerta de enlace y el dispositivo de nivel inferior se ejecutan en dispositivos físicos de la red local.

Para probar el escenario de puerta de enlace transparente, seleccione el botón siguiente para implementar dos máquinas virtuales Linux. Una máquina virtual es una puerta de enlace transparente IoT Edge, mientras que la otra es un dispositivo de nivel inferior que simula ser un termostato:

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure-Samples%2Fiot-central-docs-samples%2Fmaster%2Ftransparent-gateway%2FDeployGatewayVMs.json" target="_blank">
    <img src="https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.png" alt="Deploy to Azure button" />
</a>

Cuando se implementen y ejecuten las dos máquinas virtuales, compruebe que el dispositivo de puerta de enlace IoT Edge está en ejecución en la máquina virtual `edgegateway`:

1. Vaya a la página **Dispositivos** de la aplicación de IoT Central. Si el dispositivo de puerta de enlace IoT Edge está conectado a IoT Central, su estado es **Aprovisionado**.

1. Abra el dispositivo de puerta de enlace IoT Edge y compruebe el estado de los módulos en la página **Módulos**. Si el entorno de ejecución de Azure IoT Edge se inició correctamente, el estado de los módulos **$edgeAgent** y **$edgeHub** es **En ejecución**:

    :::image type="content" source="media/how-to-connect-iot-edge-transparent-gateway/iot-edge-runtime.png" alt-text="Captura de pantalla que muestra los módulos $edgeAgent y $edgeHub en ejecución en la puerta de enlace IoT Edge.":::

> [!TIP]
> Es posible que tenga que esperar varios minutos mientras se inicia la máquina virtual y el dispositivo se aprovisiona en la aplicación de IoT Central.

## <a name="configure-the-gateway"></a>Configuración de la puerta de enlace

Para que el dispositivo IoT Edge funcione como puerta de enlace transparente, necesita algunos certificados para demostrar su identidad en cualquier dispositivo de nivel inferior. En este artículo se utilizan certificados de demostración. En un entorno de producción, use certificados de su entidad de certificación.

Para generar los certificados de demostración e instalarlos en el dispositivo de puerta de enlace, haga lo siguiente:

1. Use SSH para conectarse e iniciar sesión en la máquina virtual del dispositivo de puerta de enlace.

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

Si el entorno de ejecución de Azure IoT Edge se inicia correctamente después de los cambios, el estado de los módulos **$edgeAgent** y **$edgeHub** cambia a **En ejecución** en la página **Módulos** del dispositivo de puerta de enlace en IoT Central.

Si el entorno de ejecución no se inicia, compruebe los cambios que realizó en *config.yaml* y consulte [Solución de problemas del dispositivo IoT Edge](../../iot-edge/troubleshoot.md).

La puerta de enlace transparente ya está configurada y lista para empezar a reenviar telemetría desde los dispositivos de nivel inferior.

## <a name="provision-a-downstream-device"></a>Aprovisionamiento de un dispositivo de nivel inferior

Actualmente, IoT Edge no puede aprovisionar de manera automática un dispositivo de nivel inferior a la aplicación de IoT Central. Estos pasos le muestran cómo aprovisionar el dispositivo `thermostat1`. Para completar estos pasos, necesita un entorno con Python 3.6 o superior instalado y conectividad de Internet. Python 3.7 viene preinstalado en [Azure Cloud Shell](https://shell.azure.com/):

1. Ejecute el comando siguiente para instalar el módulo `azure.iot.device`:

    ```bash
    pip install azure.iot.device
    ```

1. Ejecute el comando siguiente para descargar el script de Python que realiza el aprovisionamiento:

    ```bash
    wget https://raw.githubusercontent.com/Azure-Samples/iot-central-docs-samples/master/transparent-gateway/provision_device.py
    ```

1. Para aprovisionar el dispositivo de nivel inferior `thermostat1` en la aplicación de IoT Central, ejecute los comandos siguientes, reemplazando `{your application id scope}` y `{your device primary key}`:

    ```bash
    export IOTHUB_DEVICE_DPS_DEVICE_ID=thermostat1
    export IOTHUB_DEVICE_DPS_ID_SCOPE={your application id scope}
    export IOTHUB_DEVICE_DPS_DEVICE_KEY={your device primary key}
    python provision_device.py
    ```

En la aplicación de IoT Central, compruebe que el **Estado del dispositivo** thermostat1 ahora es **Aprovisionado**. 

## <a name="configure-a-downstream-device"></a>Configuración de un dispositivo de nivel inferior

En la sección anterior, configuró la máquina virtual `edgegateway` con los certificados de demostración para permitir que se ejecute como puerta de enlace. La máquina virtual `leafdevice` está lista para la instalación de un simulador de termostato que usa la puerta de enlace para conectarse a IoT Central.

La máquina virtual `leafdevice` necesita una copia del certificado de CA raíz que creó en la máquina virtual `edgegateway`. Copie el archivo */home/AzureUser/certs/certs/azure-iot-test-only.root.ca.cert.pem* de la máquina virtual `edgegateway` en el directorio principal de la máquina virtual `leafdevice`. Puede usar el comando **scp** para copiar archivos en una máquina virtual Linux y desde ella.

Consulte [Prueba de la conexión de puerta de enlace](../../iot-edge/how-to-connect-downstream-device.md#test-the-gateway-connection) para saber cómo comprobar la conexión desde el dispositivo de nivel inferior hasta la puerta de enlace.

Para ejecutar el simulador de termostato en la máquina virtual `leafdevice`, siga estos pasos:

1. Descargue el ejemplo de Python en el directorio principal:

    ```bash
    cd ~
    wget https://raw.githubusercontent.com/Azure-Samples/iot-central-docs-samples/master/transparent-gateway/simple_thermostat.py
    ```

1. Instale el módulo de Python del dispositivo Azure IoT:

    ```bash
    sudo apt update
    sudo apt install python3-pip
    pip3 install azure.iot.device
    ```

1. Establezca las variables de entorno para configurar el ejemplo. Reemplace `{your device shared key}` por la clave principal de `thermostat1` que anotó anteriormente. Estas variables suponen que el nombre de la máquina virtual de la puerta de enlace es `edgegateway` y el id. del dispositivo de termostato es `thermostat1`:

    ```bash
    export IOTHUB_DEVICE_SECURITY_TYPE=connectionString
    export IOTHUB_DEVICE_CONNECTION_STRING="HostName=edgegateway;DeviceId=thermostat1;SharedAccessKey={your device shared key}"
    export IOTEDGE_ROOT_CA_CERT_PATH=~/azure-iot-test-only.root.ca.cert.pem
    ```

    Observe que la cadena de conexión usa el nombre del dispositivo de puerta de enlace y no el de un centro de IoT.

1. Use el comando siguiente para ejecutar el código:

    ```bash
    python3 simple_thermostat.py
    ```

    La salida de este comando tiene un aspecto similar al siguiente:

    ```output
    Connecting using Connection String HostName=edgegateway;DeviceId=thermostat1;SharedAccessKey={your device shared key}
    Listening for command requests and property updates
    Press Q to quit
    Sending telemetry for temperature
    Sent message
    Sent message
    Sent message
    ...
    ```

1. Para ver la telemetría en IoT Central, vaya a la página **Información general** del dispositivo **thermostat1**:

    :::image type="content" source="media/how-to-connect-iot-edge-transparent-gateway/downstream-device-telemetry.png" alt-text="Captura de pantalla que muestra la telemetría enviada desde el dispositivo de nivel inferior.":::

    En la página **About** (Acerca de) puede ver los valores de propiedad enviados desde el dispositivo de nivel inferior y, en la página **Command** (Comando), puede llamar a comandos en el dispositivo de nivel inferior.

## <a name="next-steps"></a>Pasos siguientes

Ahora que aprendió a configurar una puerta de enlace transparente con IoT Central, el paso siguiente sugerido es aprender más sobre [IoT Edge](../../iot-edge/about-iot-edge.md).
