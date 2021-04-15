---
title: 'Errores comunes: Azure IoT Edge | Microsoft Docs'
description: Use este artículo para resolver problemas comunes que se producen al implementar una solución de IoT Edge
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 03/01/2021
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.custom:
- amqp
- mqtt
ms.openlocfilehash: 0d36a51865f3ed4a093998b16aaa174432c5308a
ms.sourcegitcommit: 02bc06155692213ef031f049f5dcf4c418e9f509
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/03/2021
ms.locfileid: "106275658"
---
# <a name="common-issues-and-resolutions-for-azure-iot-edge"></a>Problemas habituales y soluciones para Azure IoT Edge

[!INCLUDE [iot-edge-version-201806-or-202011](../../includes/iot-edge-version-201806-or-202011.md)]

Use este artículo para obtener pasos para resolver problemas comunes que pueden surgir al implementar soluciones de IoT Edge. Si necesita información sobre cómo buscar registros y errores en el dispositivo IoT Edge, consulte [Solución de problemas del dispositivo IoT Edge](troubleshoot.md).

## <a name="iot-edge-agent-stops-after-about-a-minute"></a>El agente de IoT Edge se detiene después de aproximadamente un minuto

**Comportamiento observado:**

El módulo edgeAgent se inicia y se ejecuta correctamente durante un minuto aproximadamente y luego se detiene. Los registros indican que el agente de IoT Edge intenta conectarse a IoT Hub a través de AMQP y después intenta conectarse mediante AMQP a través de WebSocket. Cuando se produce un error, se cierra el agente de IoT Edge.

Registros de ejemplo de edgeAgent:

```output
2017-11-28 18:46:19 [INF] - Starting module management agent.
2017-11-28 18:46:19 [INF] - Version - 1.0.7516610 (03c94f85d0833a861a43c669842f0817924911d5)
2017-11-28 18:46:19 [INF] - Edge agent attempting to connect to IoT Hub via AMQP...
2017-11-28 18:46:49 [INF] - Edge agent attempting to connect to IoT Hub via AMQP over WebSocket...
```

**Causa principal:**

Una configuración de redes en la red del host evita que el agente de IoT Edge alcance la red. El agente intentará conectarse en primer lugar a través de AMQP (puerto 5671). Si se produce un error en la conexión, lo intentará mediante los protocolos WebSocket (puerto 443).

El entorno de ejecución de IoT Edge configura una red para cada uno de los módulos en los que se comunica. En Linux, esta red es una red de puente. En Windows, se usa NAT. Este problema es más frecuente en los dispositivos de Windows que usan contenedores de Windows que usan la red NAT.

**Resolución:**

Asegúrese de que hay una ruta a Internet para las direcciones IP asignadas a esta red de puente o NAT. A veces, una configuración de VPN en el host invalida la red de IoT Edge.

## <a name="iot-edge-agent-cant-access-a-modules-image-403"></a>El agente de IoT Edge no puede acceder a la imagen de un módulo (403)

**Comportamiento observado:**

No se puede ejecutar un contenedor y los registros de edgeAgent muestran un error 403.

**Causa principal:**

El agente de IoT Edge no tiene permisos para acceder a la imagen de un módulo.

**Resolución:**

Asegúrese de que las credenciales del registro se hayan especificado correctamente en el manifiesto de implementación.

## <a name="edge-agent-module-reports-empty-config-file-and-no-modules-start-on-the-device"></a>El módulo Agente de Edge notifica "archivo de configuración vacío" y no se inicia ningún módulo en el dispositivo

**Comportamiento observado:**

El dispositivo tiene problemas para iniciar los módulos definidos en la implementación. Solo edgeAgent se está ejecutando, pero continuamente notifica "archivo de configuración vacío..."

**Causa principal:**

De forma predeterminada, IoT Edge inicia módulos en su propia red de contenedores aislada. El dispositivo pueda tener problemas con la resolución de nombres DNS dentro de esta red privada.

**Resolución:**

**Opción 1: establecer el servidor DNS en la configuración del motor de contenedor**

Especifique el servidor DNS del entorno en la configuración del motor de contenedor que se va a aplicar a todos los módulos de contenedor iniciados por el motor. Cree un archivo denominado `daemon.json` en el que se especifique el servidor DNS que se va a usar. Por ejemplo:

```json
{
    "dns": ["1.1.1.1"]
}
```

El ejemplo anterior establece el servidor DNS en un servicio DNS accesible públicamente. Si el dispositivo de Edge no puede acceder a esta dirección IP desde su entorno, reemplácela por una dirección de servidor DNS que sea accesible.

<!-- 1.1 -->
:::moniker range="iotedge-2018-06"
Coloque `daemon.json` en la ubicación correcta para la plataforma:

| Plataforma | Location |
| --------- | -------- |
| Linux | `/etc/docker` |
| Host de Windows con contenedores de Windows | `C:\ProgramData\iotedge-moby\config` |

Si la ubicación ya contiene el archivo `daemon.json`, agréguele la clave **dns** y guarde el archivo.

Reinicie el motor de contenedor para que las actualizaciones se apliquen.

| Plataforma | Get-Help |
| --------- | -------- |
| Linux | `sudo systemctl restart docker` |
| Windows (Administrador de PowerShell) | `Restart-Service iotedge-moby -Force` |

:::moniker-end
<!-- end 1.1 -->

<!-- 1.2 -->
:::moniker range=">=iotedge-2020-11"
Coloque `daemon.json` en el directorio `/etc/docker` del dispositivo.

Si la ubicación ya contiene un archivo `daemon.json`, agréguele la clave **dns** y guarde el archivo.

Reinicie el motor de contenedor para que las actualizaciones se apliquen.

```bash
sudo systemctl restart docker
```

:::moniker-end
<!-- end 1.2 -->

**Opción 2: establecer el servidor DNS en la implementación de IoT Edge por módulo**

Puede establecer el servidor DNS para el elemento *createOptions* de cada módulo en la implementación de IoT Edge. Por ejemplo:

```json
"createOptions": {
  "HostConfig": {
    "Dns": [
      "x.x.x.x"
    ]
  }
}
```

Asegúrese de establecer esta configuración también para los módulos *edgeAgent* y *edgeHub*.

## <a name="iot-edge-hub-fails-to-start"></a>No se puede iniciar el centro de IoT Edge

**Comportamiento observado:**

El módulo edgeHub no se inicia. Puede ver un mensaje similar a uno de los siguientes errores en los registros:

```output
One or more errors occurred.
(Docker API responded with status code=InternalServerError, response=
{\"message\":\"driver failed programming external connectivity on endpoint edgeHub (6a82e5e994bab5187939049684fb64efe07606d2bb8a4cc5655b2a9bad5f8c80):
Error starting userland proxy: Bind for 0.0.0.0:443 failed: port is already allocated\"}\n)
```

Or

```output
info: edgelet_docker::runtime -- Starting module edgeHub...
warn: edgelet_utils::logging -- Could not start module edgeHub
warn: edgelet_utils::logging --     caused by: failed to create endpoint edgeHub on network nat: hnsCall failed in Win32:  
        The process cannot access the file because it is being used by another process. (0x20)
```

**Causa principal:**

Algún otro proceso en la máquina host ha enlazado un puerto al que el módulo edgeHub está intentando enlazarse. El centro de IoT Edge asigna los puertos 443, 5671 y 8883 para su uso en escenarios de puerta de enlace. El módulo no se inicia si otro proceso ya ha enlazado uno de esos puertos.

**Resolución:**

Puede resolver este problema de dos maneras:

Si el dispositivo IoT Edge funciona como un dispositivo de puerta de enlace, debe buscar y detener el proceso que está usando el puerto 443, 5671 o 8883. Un error en el puerto 443 normalmente significa que el otro proceso es un servidor web.

Si no necesita usar el dispositivo IoT Edge como puerta de enlace, puede quitar los enlaces de puertos de las opciones de creación del módulo edgeHub. Puede cambiar las opciones de creación en Azure Portal o directamente en el archivo deployment.json.

En Azure Portal:

1. Vaya al centro de IoT y seleccione **IoT Edge**.

2. Seleccione el dispositivo IoT Edge que desee actualizar.

3. Seleccione **Set modules** (Establecer módulos).

4. Seleccione **Configuración del entorno de ejecución**.

5. En la configuración del módulo **Edge Hub**, elimine todo el contenido del cuadro de texto **Opciones de creación**.

6. Guarde los cambios y cree la implementación.

En el archivo deployment.json:

1. Abra el archivo deployment.json que aplicó al dispositivo IoT Edge.

2. Busque la configuración de `edgeHub` en la sección de propiedades deseadas de edgeAgent:

   ```json
   "edgeHub": {
       "settings": {
           "image": "mcr.microsoft.com/azureiotedge-hub:1.1",
           "createOptions": "{\"HostConfig\":{\"PortBindings\":{\"8883/tcp\":[{\"HostPort\":\"8883\"}],\"443/tcp\":[{\"HostPort\":\"443\"}]}}}"
       },
       "type": "docker",
       "status": "running",
       "restartPolicy": "always"
   }
   ```

3. Quite la línea `createOptions` y la coma al final de la línea `image` anterior:

   ```json
   "edgeHub": {
       "settings": {
           "image": "mcr.microsoft.com/azureiotedge-hub:1.1"
       },
       "type": "docker",
       "status": "running",
       "restartPolicy": "always"
   }
   ```

4. Guarde el archivo y vuelva a aplicarlo al dispositivo IoT Edge.

## <a name="iot-edge-security-daemon-fails-with-an-invalid-hostname"></a>Error de nombre de host no válido del demonio de seguridad de IoT Edge

**Comportamiento observado:**

Al intentar [consultar los registros del administrador de seguridad de IoT Edge](troubleshoot.md#check-the-status-of-the-iot-edge-security-manager-and-its-logs) se produce un error y se muestra el siguiente mensaje:

```output
Error parsing user input data: invalid hostname. Hostname cannot be empty or greater than 64 characters
```

**Causa principal:**

El tiempo de ejecución de IoT Edge solo puede admitir los nombres de host que tienen menos de 64 caracteres. Las máquinas físicas no suelen tener nombres de host largos, pero el problema es más común en una máquina virtual. Los nombres de host generados automáticamente para las máquinas virtuales Windows hospedadas en Azure, en particular, suelen ser largos.

**Resolución:**

Cuando vea este error, puede resolverlo configurando el nombre DNS de la máquina virtual y, a continuación, estableciendo el nombre DNS como nombre de host en el comando de configuración.

<!-- 1.1 -->
:::moniker range="iotedge-2018-06"

1. En Azure Portal, navegue a la hoja de introducción de la máquina virtual.
2. Seleccione **configurar** en el nombre DNS. Si la máquina virtual ya tiene configurado un nombre DNS, no es necesario configurar uno nuevo.

   ![Configuración del nombre DNS de la máquina virtual](./media/troubleshoot/configure-dns.png)

3. Proporcione un valor para **Etiqueta de nombre DNS** y seleccione **Guardar**.
4. Copie el nuevo nombre DNS, que debe tener el formato **\<DNSnamelabel\>.\<vmlocation\>.cloudapp.azure.com**.
5. Dentro de la máquina virtual, use el comando siguiente para configurar el tiempo de ejecución de IoT Edge con el nombre DNS:

   * En Linux:

      ```bash
      sudo nano /etc/iotedge/config.yaml
      ```

   * En Windows:

      ```cmd
      notepad C:\ProgramData\iotedge\config.yaml
      ```

:::moniker-end
<!-- end 1.1 -->

<!-- 1.2 -->
:::moniker range=">=iotedge-2020-11"

1. En Azure Portal, navegue a la hoja de introducción de la máquina virtual.

2. Seleccione **configurar** en el nombre DNS. Si la máquina virtual ya tiene configurado un nombre DNS, no es necesario configurar uno nuevo.

   ![Configuración del nombre DNS de la máquina virtual](./media/troubleshoot/configure-dns.png)

3. Proporcione un valor para **Etiqueta de nombre DNS** y seleccione **Guardar**.

4. Copie el nuevo nombre DNS, que debe tener el formato **\<DNSnamelabel\>.\<vmlocation\>.cloudapp.azure.com**.

5. En el dispositivo IoT Edge, abra el archivo de configuración.

   ```bash
   sudo nano /etc/aziot/config.toml
   ```

6. Reemplace el valor de `hostname` por su nombre DNS.

7. Guarde y cierre el archivo y, a continuación, aplique los cambios a IoT Edge.

   ```bash
   sudo iotedge config apply
   ```

:::moniker-end
<!-- end 1.2 -->

<!-- 1.1 -->
:::moniker range="iotedge-2018-06"

## <a name="cant-get-the-iot-edge-daemon-logs-on-windows"></a>No se obtienen los registros del demonio de IoT Edge en Windows

**Comportamiento observado:**

Aparece EventLogException al usar `Get-WinEvent` en Windows.

**Causa principal:**

El comando de PowerShell `Get-WinEvent` se basa en que esté presente una entrada del Registro para encontrar los registros por un elemento `ProviderName` concreto.

**Resolución:**

Establezca una entrada del registro para el demonio de IoT Edge. Cree un archivo **iotedge.reg** con el siguiente contenido e impórtelo en el Registro de Windows haciendo doble clic en él o mediante el comando `reg import iotedge.reg`:

```reg
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\EventLog\Application\iotedged]
"CustomSource"=dword:00000001
"EventMessageFile"="C:\\ProgramData\\iotedge\\iotedged.exe"
"TypesSupported"=dword:00000007
```

:::moniker-end
<!-- end 1.1 -->

## <a name="stability-issues-on-smaller-devices"></a>Problemas de estabilidad en dispositivos más pequeños

**Comportamiento observado:**

Es posible que experimente problemas de estabilidad en dispositivos con recursos limitados como Raspberry Pi, especialmente cuando se utiliza como puerta de enlace. Los síntomas incluyen excepciones de memoria insuficiente en el módulo del centro de IoT Edge, los dispositivos de nivel inferior no pueden conectarse o el dispositivo deja de enviar mensajes de telemetría después de unas horas.

**Causa principal:**

El centro de IoT Edge, que forma parte del runtime de IoT Edge, está optimizado para el rendimiento de manera predeterminada e intenta asignar grandes fragmentos de memoria. Esta optimización no es ideal para dispositivos con perímetro limitado y puede causar problemas de estabilidad.

**Resolución:**

En el centro de IoT Edge, establezca una variable de entorno **OptimizeForPerformance** en **false**. Hay dos maneras de establecer variables de entorno:

En Azure Portal:

En el IoT Hub, seleccione el dispositivo de IoT Edge y, en la página Detalles del dispositivo, seleccione **Establecer módulos** > **Configuración de tiempo de ejecución**. Cree una variable de entorno para el módulo del centro de IoT Edge denominada *OptimizeForPerformance* establecida en *false*.

![OptimizeForPerformance establecido en false](./media/troubleshoot/optimizeforperformance-false.png)

En el manifiesto de implementación:

```json
"edgeHub": {
  "type": "docker",
  "settings": {
    "image": "mcr.microsoft.com/azureiotedge-hub:1.1",
    "createOptions": <snipped>
  },
  "env": {
    "OptimizeForPerformance": {
      "value": "false"
    }
  },
```

## <a name="iot-edge-module-fails-to-send-a-message-to-edgehub-with-404-error"></a>El módulo de IoT Edge no puede enviar un mensaje a edgeHub y se produce el error 404

**Comportamiento observado:**

Un módulo de IoT Edge personalizado no puede enviar un mensaje al centro de IoT Edge y se produce el error 404 `Module not found`. El demonio de IoT Edge imprime el mensaje siguiente en los registros:

```output
Error: Time:Thu Jun  4 19:44:58 2018 File:/usr/sdk/src/c/provisioning_client/adapters/hsm_client_http_edge.c Func:on_edge_hsm_http_recv Line:364 executing HTTP request fails, status=404, response_buffer={"message":"Module not found"}u, 04 )
```

**Causa principal:**

El demonio de IoT Edge aplica la identificación de proceso para todos los módulos que se conectan a edgeHub por motivos de seguridad. Comprueba que todos los mensajes enviados por un módulo proceden del identificador de proceso principal del módulo. Si un módulo envía un mensaje desde un identificador de proceso diferente del que se estableció inicialmente, se rechazará el mensaje y se generará el mensaje de error 404.

**Resolución:**

A partir de la versión 1.0.7, todos los procesos de módulo tienen autorización para conectarse. Para más información, consulte el [registro de cambios de la versión v1.1.0.7](https://github.com/Azure/iotedge/blob/master/CHANGELOG.md#iotedged-1).

Si no es posible actualizar a la versión 1.0.7, realice los pasos siguientes. Asegúrese de que el módulo de IoT Edge personalizado siempre usa el mismo identificador de proceso para enviar los mensajes a edgeHub. Por ejemplo, asegúrese de usar `ENTRYPOINT` en lugar del comando `CMD` en el archivo de Docker. El comando `CMD` genera un identificador de proceso para el módulo y otro identificador de proceso para el comando bash que ejecuta el programa principal, pero `ENTRYPOINT` produce un único identificador de proceso.

## <a name="iot-edge-module-deploys-successfully-then-disappears-from-device"></a>El módulo de IoT Edge módulo se implementa correctamente y, a continuación, desaparece del dispositivo

**Comportamiento observado:**

Después de configurar los módulos para un dispositivo IoT Edge, los módulos se implementan correctamente, pero después de unos minutos desaparecen del dispositivo y de los detalles del dispositivo en Azure Portal. También pueden aparecer en el dispositivo otros módulos distintos a los definidos.

**Causa principal:**

Si una implementación automática tiene como destino un dispositivo, tendrá prioridad sobre la configuración manual de los módulos para un único dispositivo. La funcionalidad **Establecer módulos** de Azure Portal o la funcionalidad **Crear una implementación para un dispositivo individual** de Visual Studio Code se aplicarán momentáneamente. Inicialmente, verá en el dispositivo los módulos que ha definido. A continuación, se aplicará la prioridad de la implementación automática, que sobrescribe las propiedades deseadas del dispositivo.

**Resolución:**

Use solo un tipo de mecanismo de implementación por dispositivo, ya sea una implementación automática o implementaciones de dispositivo individuales. Si tiene varias implementaciones automáticas que tienen como destino un dispositivo, puede cambiar la prioridad o las descripciones de destino para asegurarse de que se aplique la correcta a un dispositivo determinado. También puede actualizar el dispositivo gemelo para que ya no coincida con la descripción de destino de la implementación automática.

Para más información, consulte el artículo [Descripción de las implementaciones automáticas de IoT Edge en un único dispositivo o a escala](module-deployment-monitoring.md).

<!-- <1.2> -->
::: moniker range=">=iotedge-2020-11"

## <a name="iot-edge-behind-a-gateway-cannot-perform-http-requests-and-start-edgeagent-module"></a>IoT Edge detrás de una puerta de enlace no puede realizar solicitudes HTTP ni iniciar el módulo edgeAgent

**Comportamiento observado:**

el demonio de IoT Edge está activo con un archivo de configuración válido, pero no puede iniciar el módulo edgeAgent. El comando `iotedge list` devuelve una lista vacía. El demonio de IoT Edge registra el informe `Could not perform HTTP request`.

**Causa principal:**

los dispositivos de IoT Edge detrás de una puerta de enlace obtienen las imágenes del módulo del dispositivo de IoT Edge primario especificado en el campo `parent_hostname` del archivo de configuración. El error `Could not perform HTTP request` significa que el dispositivo secundario no puede llegar a su dispositivo primario a través de HTTP.

**Solución:**

asegúrese de que el dispositivo de IoT Edge primario puede recibir solicitudes entrantes del dispositivo de IoT Edge secundario. Abra el tráfico de red en los puertos 443 y 6617 para las solicitudes procedentes del dispositivo secundario.

:::moniker-end
<!-- end 1.2 -->

## <a name="next-steps"></a>Pasos siguientes

¿Cree que encontró un error en la plataforma de IoT Edge? [Envíe un problema](https://github.com/Azure/iotedge/issues) para que podamos seguir mejorando.

Si tiene más preguntas, cree una [solicitud de soporte técnico](https://portal.azure.com/#create/Microsoft.Support) para obtener ayuda.
