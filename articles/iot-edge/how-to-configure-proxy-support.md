---
title: 'Configuración de dispositivos para servidores proxy de red: Azure IoT Edge | Microsoft Docs'
description: Aprenda a configurar el entorno de ejecución de Azure IoT Edge y todos los módulos de IoT Edge a los que se pueda acceder desde Internet para que se comuniquen a través de un servidor proxy.
author: kgremban
ms.author: kgremban
ms.date: 09/03/2020
ms.topic: how-to
ms.service: iot-edge
services: iot-edge
ms.custom:
- amqp
- contperf-fy21q1
ms.openlocfilehash: 9f2ca089a6d885227bd61940d71ec7bb7960fbd6
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105043162"
---
# <a name="configure-an-iot-edge-device-to-communicate-through-a-proxy-server"></a>Configuración de un dispositivo de IoT Edge para que se comunique a través de un servidor proxy

[!INCLUDE [iot-edge-version-201806-or-202011](../../includes/iot-edge-version-201806-or-202011.md)]

Los dispositivos de IoT Edge envían solicitudes HTTPS para comunicarse con IoT Hub. Si el dispositivo está conectado a una red que usa un servidor proxy, deberá configurar el entorno de ejecución de IoT Edge para que se comunique a través del servidor. Los servidores proxy también pueden afectar a módulos de IoT Edge individuales si estos realizan solicitudes HTTP o HTTPS que no se enrutan a través del centro de IoT Edge.

Este artículo le guiará por los cuatro pasos que se describen a continuación para configurar y administrar un dispositivo de IoT Edge situado detrás de un servidor proxy:

1. [**Instale el entorno de ejecución de Azure IoT Edge en el dispositivo.**](#install-iot-edge-through-a-proxy)

   Los scripts de instalación de IoT Edge extraen paquetes y archivos de Internet, por lo que es necesario que la comunicación con el dispositivo pueda atravesar el servidor proxy para realizar esas solicitudes. En el caso de los dispositivos Windows, el script de instalación también dispone de una opción de instalación sin conexión.

   Este paso es un proceso que solamente tiene que realizarse una vez para configurar el dispositivo de IoT Edge al establecer la configuración inicial. Estas mismas conexiones también son necesarias para actualizar el entorno de ejecución de IoT Edge.

2. [**Configuración de IoT Edge y el tiempo de ejecución de contenedor en el dispositivo**](#configure-iot-edge-and-moby)

   IoT Edge es responsable de las comunicaciones con IoT Hub. El tiempo de ejecución de contenedor es responsable de la administración de contenedores, por lo que se comunica con los registros del contenedor. Ambos componentes deben hacer las solicitudes web a través del servidor proxy.

   Este paso es un proceso que solamente tiene que realizarse una vez para configurar el dispositivo de IoT Edge al establecer la configuración inicial.

3. [**Configuración de las propiedades del agente de IoT Edge en el archivo de configuración del dispositivo**](#configure-the-iot-edge-agent)

   El demonio de IoT Edge inicia el módulo edgeAgent inicialmente. Después, el módulo edgeAgent recupera el manifiesto de implementación de IoT Hub e inicia todos los demás módulos. Para que el agente de IoT Edge realice la conexión inicial con IoT Hub, configure manualmente las variables de entorno del módulo edgeAgent en el propio dispositivo. Una vez establecida la conexión inicial, puede configurar el módulo de edgeAgent de forma remota.

   Este paso es un proceso que solamente tiene que realizarse una vez para configurar el dispositivo de IoT Edge al establecer la configuración inicial.

4. [**En el futuro, siempre que implemente un módulo, defina las variables de entorno de todos los módulos que se comuniquen a través del proxy.**](#configure-deployment-manifests)

   Una vez que el dispositivo IoT Edge esté configurado y conectado a IoT Hub a través del servidor proxy, deberá mantener la conexión en todas las implementaciones de módulos que se realicen en el futuro.

   Este paso es un proceso continuo que se realiza de forma remota para que, con cada nueva actualización del módulo o la implementación, el dispositivo conserve su capacidad para comunicarse a través del servidor proxy.

## <a name="know-your-proxy-url"></a>Dirección URL de proxy

Para poder comenzar los pasos que se describen en este artículo, debe conocer la dirección URL del proxy.

Las direcciones URL de proxy tienen el formato siguiente: **protocolo**://**host_proxy**:**puerto_proxy**.

* El **protocolo** es HTTP o HTTPS. El demonio de Docker puede usar cualquier protocolo según la configuración del registro de contenedor, pero los contenedores en runtime y demonio de IoT Edge deben utilizar siempre HTTPS para conectarse al proxy.

* El **host_proxy** es una dirección para el servidor proxy. Si el servidor proxy requiere autenticación, puede proporcionar sus credenciales como parte del host proxy utilizando el siguiente formato: **usuario**:**contraseña**\@**host_proxy**.

* El **puerto_proxy** es el puerto de red en el que el proxy responde al tráfico de red.

## <a name="install-iot-edge-through-a-proxy"></a>Instalación de IoT Edge a través de un proxy

Tanto si el dispositivo de IoT Edge se ejecuta en Windows como si se ejecuta en Linux, debe obtener acceso a los paquetes de instalación a través del servidor proxy. En función del sistema operativo, siga el procedimiento que corresponda para instalar el entorno de ejecución de IoT Edge a través de un servidor proxy.

### <a name="linux-devices"></a>Dispositivos Linux

Si va a instalar el entorno de ejecución de Azure IoT Edge en un dispositivo Linux, configure el administrador de paquetes para que pase por el servidor proxy para acceder al paquete de instalación. Por ejemplo, [configure apt-get para usar un http-proxy](https://help.ubuntu.com/community/AptGet/Howto/#Setting_up_apt-get_to_use_a_http-proxy). Una vez que el administrador de paquetes esté configurado, siga las instrucciones de [Instalación del entorno de ejecución de Azure IoT Edge](how-to-install-iot-edge.md) como de costumbre.

### <a name="windows-devices-using-iot-edge-for-linux-on-windows"></a>Dispositivos de Windows con IoT Edge para Linux en Windows.

Si va a instalar el entorno de ejecución de IoT Edge mediante IoT Edge para Linux en Windows, IoT Edge se instala de forma predeterminada en la máquina virtual de Linux. No se requieren pasos adicionales de instalación o actualización.

### <a name="windows-devices-using-windows-containers"></a>Dispositivos de Windows con contenedores de Windows

Si va a instalar el entorno de ejecución de Azure IoT Edge en un dispositivo Windows, debe pasar por el servidor proxy dos veces. La primera conexión descarga el archivo del script de instalación. La segunda conexión se produce durante la instalación para descargar los componentes necesarios. Puede especificar la información del proxy en la configuración de Windows o incluirla directamente en los comandos de PowerShell.

Los siguientes pasos son un ejemplo de una instalación de Windows con el argumento `-proxy`:

1. El comando Invoke-WebRequest necesita información del proxy para poder acceder al script de instalación. A continuación, el comando Deploy-IoTEdge necesita la información del proxy para poder descargar los archivos de instalación.

   ```powershell
   . {Invoke-WebRequest -proxy <proxy URL> -useb aka.ms/iotedge-win} | Invoke-Expression; Deploy-IoTEdge -proxy <proxy URL>
   ```

2. El comando Initialize-IoTEdge no tiene que atravesar el proxy, por lo que el segundo paso solo necesita la información del proxy para Invoke-WebRequest.

   ```powershell
   . {Invoke-WebRequest -proxy <proxy URL> -useb aka.ms/iotedge-win} | Invoke-Expression; Initialize-IoTEdge
   ```

Si tiene credenciales de servidor proxy demasiado complejas como para incluirlas en la dirección URL, use el parámetro `-ProxyCredential` en el comando `-InvokeWebRequestParameters`. Por ejemplo,

```powershell
$proxyCredential = (Get-Credential).GetNetworkCredential()
. {Invoke-WebRequest -proxy <proxy URL> -ProxyCredential $proxyCredential -useb aka.ms/iotedge-win} | Invoke-Expression; `
Deploy-IoTEdge -InvokeWebRequestParameters @{ '-Proxy' = '<proxy URL>'; '-ProxyCredential' = $proxyCredential }
```

Para obtener más información acerca de los parámetros de proxy, consulte [Invoke-WebRequest](/powershell/module/microsoft.powershell.utility/invoke-webrequest). Para obtener más información sobre los parámetros de instalación de Windows, consulte [Scripts de PowerShell para IoT Edge en Windows](reference-windows-scripts.md).

## <a name="configure-iot-edge-and-moby"></a>Configuración de IoT Edge y Moby

IoT Edge utiliza dos demonios que se ejecutan en el dispositivo de IoT Edge. El demonio de Docker realiza solicitudes web para extraer imágenes de contenedor desde los registros de contenedor. El demonio de IoT Edge realiza solicitudes web para comunicarse con IoT Hub.

Tanto el demonio Moby como el demonio IoT Edge tienen que configurarse de forma que utilicen el servidor proxy para garantizar la funcionalidad del dispositivo sin interrupciones. Este paso tiene lugar en el dispositivo de IoT Edge durante la instalación inicial del dispositivo.

### <a name="moby-daemon"></a>Demonio Moby

Como Moby se basa en Docker, consulte la documentación de Docker para configurar el demonio Moby con variables de entorno. La mayoría de los registros de contenedor (incluidos los de DockerHub y Azure Container) admiten solicitudes HTTPS, por lo que el parámetro que se debe establecer es **HTTPS_PROXY**. Si va a extraer imágenes de un registro que no admite la seguridad de la capa de transporte (TLS), debe establecer el parámetro **HTTP_PROXY**.

Elija el artículo que corresponda al sistema operativo del dispositivo de IoT Edge:

* [Configure Docker daemon on Linux](https://docs.docker.com/config/daemon/systemd/#httphttps-proxy) (Configuración del demonio de Docker en Linux). El demonio Moby en Linux mantiene el nombre Docker.
* [Configure Docker daemon on Windows](/virtualization/windowscontainers/manage-docker/configure-docker-daemon#proxy-configuration) (Configuración del demonio de Docker en Windows). El demonio Moby en Windows se llama iotedge-moby. Los nombres son diferentes porque, en los dispositivos Windows, Docker Desktop y Moby se pueden ejecutar a la vez.

### <a name="iot-edge-daemon"></a>Demonio de IoT Edge

El demonio de IoT Edge se configura de forma similar al de Moby. Utilice los pasos siguientes para establecer una variable de entorno para el servicio según su sistema operativo.

El demonio de IoT Edge siempre utiliza HTTPS para enviar solicitudes a IoT Hub.

#### <a name="linux"></a>Linux

<!-- 1.1 -->
:::moniker range="iotedge-2018-06"

Abra un editor en el terminal para configurar el demonio de IoT Edge.

```bash
sudo systemctl edit iotedge
```

Escriba el texto siguiente, pero reemplace **\<proxy URL>** por la dirección del servidor proxy y el puerto. A continuación, guarde y salga.

```ini
[Service]
Environment="https_proxy=<proxy URL>"
```

Actualice el administrador de servicios para que incluya la nueva configuración de IoT Edge.

```bash
sudo systemctl daemon-reload
```

Reinicie IoT Edge para que los cambios surtan efecto.

```bash
sudo systemctl restart iotedge
```

Compruebe que se creó la variable de entorno y que se cargó la nueva configuración.

```bash
systemctl show --property=Environment iotedge
```
:::moniker-end
<!--end 1.1-->

<!-- 1.2 -->
:::moniker range=">=iotedge-2020-11"

Abra un editor en el terminal para configurar el demonio de IoT Edge.

```bash
sudo systemctl edit aziot-edged
```

Escriba el texto siguiente, pero reemplace **\<proxy URL>** por la dirección del servidor proxy y el puerto. A continuación, guarde y salga.

```ini
[Service]
Environment="https_proxy=<proxy URL>"
```

A partir de la versión 1.2, IoT Edge usa el servicio de identidad de IoT para controlar el aprovisionamiento de dispositivos con IoT Hub o IoT Hub Device Provisioning Service. Abra un editor en el terminal para configurar el demonio del servicio de identidad de IoT.

```bash
sudo systemctl edit aziot-identityd
```

Escriba el texto siguiente, pero reemplace **\<proxy URL>** por la dirección del servidor proxy y el puerto. A continuación, guarde y salga.

```ini
[Service]
Environment="https_proxy=<proxy URL>"
```

Actualice el administrador de servicios para que incluya las nuevas configuraciones.

```bash
sudo systemctl daemon-reload
```

Reinicie los servicios del sistema de IoT Edge para que los cambios en ambos demonios surtan efecto.

```bash
sudo iotedge system restart
```

Compruebe que se crearon las variables de entorno y que se cargó la nueva configuración.

```bash
systemctl show --property=Environment aziot-edged
systemctl show --property=Environment aziot-identityd
```
:::moniker-end
<!--end 1.2-->

#### <a name="windows-using-iot-edge-for-linux-on-windows"></a>Windows con IoT Edge para Linux en Windows.

Inicie sesión en su máquina virtual de IoT Edge para Linux en Windows:

```azurepowershell-interactive
Ssh-EflowVm
```

Siga los mismos pasos que en la sección de Linux anterior para configurar el demonio de IoT Edge.

#### <a name="windows-using-windows-containers"></a>Windows con contenedores de Windows

Abra una ventana de PowerShell como administrador y ejecute el comando siguiente para editar el registro con la nueva variable de entorno. Reemplace **\<proxy url>** por la dirección del servidor proxy y el puerto.

```powershell
reg add HKLM\SYSTEM\CurrentControlSet\Services\iotedge /v Environment /t REG_MULTI_SZ /d https_proxy=<proxy URL>
```

Reinicie IoT Edge para que los cambios surtan efecto.

```powershell
Restart-Service iotedge
```

## <a name="configure-the-iot-edge-agent"></a>Configuración del agente de IoT Edge

El agente de IoT Edge es el primer módulo que se inicia en cualquier dispositivo de IoT Edge. Se inicia por primera vez en función de la información del archivo de configuración de IoT Edge. El agente de IoT Edge se conecta posteriormente a IoT Hub para recuperar los manifiestos de implementación, los cuales declaran qué otros módulos deberían estar implementados en el dispositivo.

Este paso se realiza una sola vez en el dispositivo de IoT Edge durante la instalación inicial del dispositivo.

<!-- 1.1 -->
:::moniker range="iotedge-2018-06"

1. Abra el archivo config.yaml en el dispositivo de IoT Edge. En los sistemas Linux, este archivo se encuentra en **/etc/iotedge/config.yaml**. En los sistemas Windows, este archivo se encuentra en **C:\ProgramData\iotedge\config.yaml**. El archivo de configuración está protegido, por lo que necesita privilegios de administrador para acceder a él. En los sistemas Linux, utilice el comando `sudo` antes de abrir el archivo en el editor de texto que prefiera. En Windows, abra como administrador un editor de texto (por ejemplo, el Bloc de notas) y después abra el archivo.

2. En el archivo config.yaml, busque la sección **especificación del módulo de agente de Edge**. La definición del agente de IoT Edge incluye un parámetro **env** en el que puede agregar variables de entorno.

3. Elimine las llaves que son marcadores de posición del parámetro env y agregue la nueva variable en una nueva línea. Recuerde que las sangrías de YAML son dos espacios.

   ```yaml
   https_proxy: "<proxy URL>"
   ```

4. El runtime de IoT Edge usa AMQP de forma predeterminada para comunicarse con IoT Hub. Algunos servidores proxy bloquean los puertos AMQP. Si es así, también debe configurar edgeAgent para que use AMQP sobre WebSocket. Agregue una segunda variable de entorno.

   ```yaml
   UpstreamProtocol: "AmqpWs"
   ```

   ![definición de edgeAgent con variables de entorno](./media/how-to-configure-proxy-support/edgeagent-edited.png)

5. Guarde los cambios en config.yaml y cierre el editor. Reinicie IoT Edge para que los cambios surtan efecto.

   * Linux e IoT Edge para Linux en Windows:

      ```bash
      sudo systemctl restart iotedge
      ```

   * Windows con contenedores de Windows:

      ```powershell
      Restart-Service iotedge
      ```

:::moniker-end
<!-- end 1.1 -->

<!-- 1.2 -->
:::moniker range=">=iotedge-2020-11"

1. Abra el archivo de configuración en el dispositivo IoT Edge: `/etc/aziot/config.toml`. El archivo de configuración está protegido, por lo que necesita privilegios de administrador para acceder a él. En los sistemas Linux, utilice el comando `sudo` antes de abrir el archivo en el editor de texto que prefiera.

2. En el archivo de configuración, busque la sección `[agent]`, que contiene toda la información de configuración para el módulo edgeAgent que se va a usar en el inicio. La definición del agente de IoT Edge incluye una subsección `[agent.env]` en la que puede agregar variables de entorno.

3. Agregue el parámetro **https_proxy** a la sección de variables de entorno y establezca la dirección URL del proxy como su valor.

   ```toml
   [agent.env]
   # "RuntimeLogLevel" = "debug"
   # "UpstreamProtocol" = "AmqpWs"
   "https_proxy" = "<proxy URL>"
   ```

4. El runtime de IoT Edge usa AMQP de forma predeterminada para comunicarse con IoT Hub. Algunos servidores proxy bloquean los puertos AMQP. Si es así, también debe configurar edgeAgent para que use AMQP sobre WebSocket. Quite la marca de comentario del parámetro `UpstreamProtocol`.

   ```toml
   [agent.env]
   # "RuntimeLogLevel" = "debug"
   "UpstreamProtocol" = "AmqpWs"
   "https_proxy" = "<proxy URL>"
   ```

5. Guarde los cambios y cierre el editor. Aplique los cambios más recientes.

   ```bash
   sudo iotedge config apply
   ```

:::moniker-end
<!-- end 1.2 -->

## <a name="configure-deployment-manifests"></a>Configuración de manifiestos de implementación  

Una vez que el dispositivo de IoT Edge esté configurado para funcionar con el servidor proxy, deberá declarar la variable de entorno HTTPS_PROXY en los manifiestos de implementación futuros. Puede editar los manifiestos de implementación utilizando el asistente de Azure Portal o editando el archivo JSON de un manifiesto de implementación.

Configure siempre los dos módulos en tiempo de ejecución (edgeAgent y edgeHub) para que se comuniquen a través del servidor proxy con el fin de que puedan mantener una conexión con IoT Hub. Si quita la información de proxy del módulo edgeAgent, la única forma de restablecer la conexión es editando el archivo de configuración en el dispositivo, tal como se describe en la sección anterior.

Además de los módulos edgeAgent y edgeHub, es posible que otros módulos necesiten la configuración del proxy. Los módulos que necesitan acceder a recursos de Azure además de IoT Hub, como el almacenamiento de blobs, deben tener la variable HTTPS_PROXY especificada en el archivo de manifiesto de implementación.

El siguiente procedimiento es aplicable a lo largo de la vida del dispositivo IoT Edge.

### <a name="azure-portal"></a>Azure portal

Cuando usa el asistente de **Establecimiento de módulos** para crear implementaciones para dispositivos de IoT Edge, todos los módulos tienen una sección **Variables de entorno** donde puede configurar las conexiones con el servidor proxy.

Para configurar el agente de IoT Edge y los módulos del centro de IoT Edge, seleccione **Configuración del entorno de ejecución** en el primer paso del asistente.

![Configurar las opciones avanzadas del entorno en tiempo de ejecución de Edge](./media/how-to-configure-proxy-support/configure-runtime.png)

Agregue la variable de entorno **https_proxy** al agente de IoT Edge y a las definiciones de módulo del centro de IoT Edge. Si incluyó la variable de entorno **UpstreamProtocol** en el archivo de configuración del dispositivo IoT Edge, agréguela también a la definición del módulo del agente de IoT Edge.

![Establecimiento de la variable de entorno https_proxy](./media/how-to-configure-proxy-support/edgehub-environmentvar.png)

Todos los demás módulos que agregue a un manifiesto de implementación seguirán el mismo patrón.

### <a name="json-deployment-manifest-files"></a>Archivos JSON del manifiesto de implementación

Si va a crear implementaciones para dispositivos IoT Edge mediante las plantillas de Visual Studio Code o mediante la creación manual de archivos JSON, puede agregar las variables de entorno directamente a cada definición del módulo.

Use el siguiente formato JSON:

```json
"env": {
    "https_proxy": {
        "value": "<proxy URL>"
    }
}
```

Con las variables de entorno incluidas, la definición del módulo debería ser similar al siguiente ejemplo de edgeHub:

```json
"edgeHub": {
    "type": "docker",
    "settings": {
        "image": "mcr.microsoft.com/azureiotedge-hub:1.1",
        "createOptions": ""
    },
    "env": {
        "https_proxy": {
            "value": "http://proxy.example.com:3128"
        }
    },
    "status": "running",
    "restartPolicy": "always"
}
```

Si incluyó la variable de entorno **UpstreamProtocol** en el archivo confige.yaml del dispositivo de IoT Edge, agréguela también a la definición del módulo del agente de IoT Edge.

```json
"env": {
    "https_proxy": {
        "value": "<proxy URL>"
    },
    "UpstreamProtocol": {
        "value": "AmqpWs"
    }
}
```

## <a name="working-with-traffic-inspecting-proxies"></a>Trabajo con proxies de inspección de tráfico

Si el proxy que intenta usar realiza inspección de tráfico en conexiones protegidas por TLS, es importante tener en cuenta que la autenticación con certificados X.509 no funciona. IoT Edge establece un canal TLS cifrado de un extremo a otro con el certificado y la clave proporcionados. Si ese canal se interrumpe para la inspección de tráfico, el proxy no puede restablecerlo con las credenciales correctas e IoT Hub y el servicio de aprovisionamiento de dispositivos de IoT Hub devuelven un error `Unauthorized`.

Para usar un proxy que realice inspección de tráfico, debe emplear autenticación de firma de acceso compartido o tener IoT Hub y el servicio de aprovisionamiento de dispositivos de IoT Hub agregados a una lista de permitidos para evitar la inspección.

## <a name="next-steps"></a>Pasos siguientes

Más información sobre los roles del [entorno de ejecución de Azure IoT Edge](iot-edge-runtime.md).

Solución de errores de instalación y configuración con [Problemas habituales y soluciones para Azure IoT Edge](troubleshoot.md)
