---
title: 'Preguntas frecuentes sobre la solución de factoría conectada: Azure | Microsoft Docs'
description: Este artículo responde a las preguntas más frecuentes sobre el acelerador de soluciones de factoría conectada. Incluye vínculos al repositorio de GitHub.
author: dominicbetts
manager: timlt
ms.service: iot-accelerators
services: iot-accelerators
ms.topic: conceptual
ms.date: 12/12/2017
ms.author: dobett
ms.openlocfilehash: 038403743caf13087655066f4acbec4dcee598c7
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107874214"
---
# <a name="frequently-asked-questions-for-connected-factory-solution-accelerator"></a>Preguntas frecuentes sobre el acelerador de la solución de factoría conectada

Consulte, además, las [preguntas más frecuentes](iot-accelerators-faq.md) sobre aceleradores de soluciones de IoT.

### <a name="where-can-i-find-the-source-code-for-the-solution-accelerator"></a>¿Dónde se puede encontrar el código fuente del acelerador de la solución?

El código fuente se almacena en el siguiente repositorios de GitHub:

* [Acelerador de la solución de factoría conectada](https://github.com/Azure/azure-iot-connected-factory)

### <a name="what-is-opc-ua"></a>¿Qué es OPC UA?

OPC Unified Architecture (UA), publicado en 2008, es un estándar de interoperabilidad independiente de plataforma y orientado a servicios. OPC UA se usa en diversos sistemas industriales y dispositivos como equipos, PLC y sensores del sector. OPC UA integra la funcionalidad de las especificaciones de OPC Classic en un solo marco extensible con seguridad integrada. Es un estándar controlado por OPC Foundation. [OPC Foundation](https://opcfoundation.org/) es una organización sin ánimo de lucro con más de 440 miembros. El objetivo de la organización es usar la especificación OPC para facilitar la interoperabilidad multiproveedor, multiplataforma, segura y confiable gracias a lo siguiente:

* Infraestructura
* Especificaciones
* Technology
* Procesos

### <a name="why-did-microsoft-choose-opc-ua-for-the-connected-factory-solution-accelerator"></a>¿Por qué Microsoft ha elegido a OPC UA para el acelerador de la solución de factoría conectada?

Microsoft ha elegido OPC UA porque es un estándar abierto, independiente de la plataforma, no es de propiedad, es reconocido por el sector y está probado. Es un requisito para las soluciones de arquitectura de referencia de Industrie 4.0 (RAMI4.0), que garantiza la interoperabilidad entre un amplio conjunto de procesos y equipos de fabricación. Microsoft ha detectado una demanda por parte de sus clientes para crear soluciones de Industrie 4.0. La compatibilidad con OPC UA ayuda a reducir las barreras para que los clientes puedan lograr sus objetivos y les proporciona un valor de negocio inmediato.

### <a name="how-do-i-add-a-public-ip-address-to-the-simulation-vm"></a>¿Cómo se agrega una dirección IP pública a la máquina virtual de simulación?

Tiene dos opciones para agregar la dirección IP:

* Usar el script de PowerShell `Simulation/Factory/Add-SimulationPublicIp.ps1` en el [repositorio](https://github.com/Azure/azure-iot-connected-factory). Pase el nombre de la implementación como un parámetro. Para una implementación local, use `<your username>ConnFactoryLocal`. El script imprime la dirección IP de la máquina virtual.

* En Azure Portal, busque el grupo de recursos de la implementación. Excepto para una implementación local, el grupo de recursos tiene el nombre especificado como nombre de solución o de implementación. Para una implementación local mediante el script de compilación, el nombre del grupo de recursos es `<your username>ConnFactoryLocal`. Ahora agregue un nuevo recurso de la **dirección IP pública** al grupo de recursos.

> [!NOTE]
> En cualquier caso, asegúrese de instalar las revisiones más recientes; para ello, siga las instrucciones que aparecen el [sitio web de Ubuntu](https://wiki.ubuntu.com/Security/Upgrades). Mantenga la instalación actualizada siempre que la máquina virtual sea accesible a través de una dirección IP pública.

### <a name="how-do-i-remove-the-public-ip-address-to-the-simulation-vm"></a>¿Cómo se quita una dirección IP pública de la máquina virtual de simulación?

Tiene dos opciones para quitar la dirección IP:

* Use el script de PowerShell Simulation/Factory/Remove-SimulationPublicIp.ps1 del [repositorio](https://github.com/Azure/azure-iot-connected-factory). Pase el nombre de la implementación como un parámetro. Para una implementación local, use `<your username>ConnFactoryLocal`. El script imprime la dirección IP de la máquina virtual.

* En Azure Portal, busque el grupo de recursos de la implementación. Excepto para una implementación local, el grupo de recursos tiene el nombre especificado como nombre de solución o de implementación. Para una implementación local mediante el script de compilación, el nombre del grupo de recursos es `<your username>ConnFactoryLocal`. Quite ahora el recurso de la **dirección IP pública** del grupo de recursos.

### <a name="how-do-i-sign-in-to-the-simulation-vm"></a>¿Cómo se inicia sesión en la máquina virtual de simulación?

Solo se admite el inicio de sesión en la máquina virtual de simulación si ha implementado la solución con el script de PowerShell `build.ps1` en el [repositorio](https://github.com/Azure/azure-iot-connected-factory).

Si ha implementado la solución de www.azureiotsolutions.com, no puede iniciar sesión en la máquina virtual. No se puede iniciar sesión porque la contraseña se genera aleatoriamente y no se puede restablecer.

1. Agregue una dirección IP pública a la máquina virtual. Consulte [¿Cómo se agrega una dirección IP pública a la máquina virtual de simulación](#how-do-i-remove-the-public-ip-address-to-the-simulation-vm)
1. Cree una sesión de SSH para la máquina virtual con la dirección IP de la máquina virtual.
1. Es el nombre de usuario que se va a utilizar es: `docker`.
1. La contraseña que se utilice depende de la versión que se usó en la implementación:
    * Para las soluciones implementadas con el script build.ps1 antes del 1 de junio de 2017, la contraseña es: `Passw0rd`.
    * Para las soluciones implementadas con el script build.ps1 después del 1 de junio de 2017, la contraseña es: `<name of your deployment>.config.user`. La contraseña se almacena en el valor **VmAdminPassword**. La contraseña se genera aleatoriamente durante la implementación a menos que la especifique con el parámetro `-VmAdminPassword` del script `build.ps1`.

### <a name="how-do-i-stop-and-start-all-docker-processes-in-the-simulation-vm"></a>¿Cómo se puede detener e iniciar todos los procesos de docker en la máquina virtual de simulación?

1. Inicie sesión en la máquina virtual de simulación. Consulte [¿Cómo se inicia sesión en la máquina virtual de simulación?](#how-do-i-sign-in-to-the-simulation-vm)
1. Para comprobar qué contenedores están activos, ejecute: `docker ps`.
1. Para detener todos los contenedores de simulación, ejecute: `./stopsimulation`.
1. Para iniciar todos los contenedores de simulación:
    * Exporte una variable de shell con el nombre **IOTHUB_CONNECTIONSTRING**. Utilice el valor de la configuración **IotHubOwnerConnectionString** en el archivo `<name of your deployment>.config.user`. Por ejemplo:

        ```sh
        export IOTHUB_CONNECTIONSTRING="HostName={yourdeployment}.azure-devices.net;SharedAccessKeyName=iothubowner;SharedAccessKey={your key}"
        ```

    * Ejecute `./startsimulation`.

### <a name="how-do-i-update-the-simulation-in-the-vm"></a>¿Cómo se actualiza la simulación en la máquina virtual?

Si ha realizado algún cambio en la simulación, puede usar el script de PowerShell `build.ps1` en el [repositorio](https://github.com/Azure/azure-iot-connected-factory) mediante el comando `updatedimulation`. Este script crea todos los componentes de simulación, detiene la simulación en la máquina virtual, realiza la carga, la instalación y los inicia.

### <a name="how-do-i-find-out-the-connection-string-of-the-iot-hub-used-by-my-solution"></a>¿Cómo se puede averiguar la cadena de conexión de IoT Hub utilizada por la solución?

Si ha implementado la solución con el script `build.ps1` en el [repositorio](https://github.com/Azure/azure-iot-connected-factory), la cadena de conexión es el valor de **IotHubOwnerConnectionString** en el archivo `<name of your deployment>.config.user`.

También puede encontrar la cadena de conexión mediante Azure Portal: En el recurso de IoT Hub del grupo de recursos de la implementación, busque el valor de la cadena de conexión.

### <a name="which-iot-hub-devices-does-the-connected-factory-simulation-use"></a>¿Qué dispositivos de IoT Hub usa la simulación de factoría conectada?

La misma simulación registra los siguientes dispositivos:

* proxy.beijing.corp.contoso
* proxy.capetown.corp.contoso
* proxy.mumbai.corp.contoso
* proxy.munich0.corp.contoso
* proxy.rio.corp.contoso
* proxy.seattle.corp.contoso
* publisher.beijing.corp.contoso
* publisher.capetown.corp.contoso
* publisher.mumbai.corp.contoso
* publisher.munich0.corp.contoso
* publisher.rio.corp.contoso
* publisher.seattle.corp.contoso

Mediante la herramienta [DeviceExplorer](https://github.com/Azure/azure-iot-sdk-csharp/tree/master/tools/) o [la extensión de IoT para la CLI de Azure](https://github.com/Azure/azure-iot-cli-extension), puede comprobar qué dispositivos están registrados con la instancia de IoT Hub que la solución usa. Para usar el explorador de dispositivos, necesita la cadena de conexión para la instancia de IoT Hub en la implementación. Para usar la extensión de IoT para la CLI de Azure, necesita el nombre de la instancia de IoT Hub.

### <a name="how-can-i-get-log-data-from-the-simulation-components"></a>¿Cómo se pueden obtener datos de registro de los componentes de simulación?

Todos los componentes de la simulación registran información en los archivos de registro. Estos archivos pueden encontrarse en la máquina virtual, en la carpeta `home/docker/Logs`. Para recuperar los registros, puede usar el script de PowerShell `Simulation/Factory/Get-SimulationLogs.ps1` en el [repositorio](https://github.com/Azure/azure-iot-connected-factory).

Este script debe iniciar sesión en la máquina virtual. Debe proporcionar credenciales para el inicio de sesión. Consulte [¿Cómo se inicia sesión en la máquina virtual de simulación?](#how-do-i-sign-in-to-the-simulation-vm) para buscar las credenciales.

El script agrega o quita una dirección IP pública en la máquina virtual, si todavía no tiene una y la quita. El script coloca todos los archivos de registro en un archivo y lo descarga en la estación de trabajo de desarrollo.

También puede iniciar sesión en la máquina virtual a través de SSH e inspeccionar los archivos de registro en tiempo de ejecución.

### <a name="how-can-i-check-if-the-simulation-is-sending-data-to-the-cloud"></a>¿Cómo puedo comprobar si la simulación envía datos a la nube?

Con [Azure IoT Explorer](https://github.com/Azure/azure-iot-explorer) o con el comando [Azure IoT CLI Extension monitor-events](/cli/azure/iot/hub#az_iot_hub_monitor_events), puede inspeccionar los datos enviados a IoT Hub desde determinados dispositivos. Para usar estas herramientas, tiene que saber la cadena de conexión para la instancia de IoT Hub de la implementación. Consulte [¿Cómo se puede averiguar la cadena de conexión de IoT Hub utilizada por la solución?](#how-do-i-find-out-the-connection-string-of-the-iot-hub-used-by-my-solution)

Inspeccione los datos enviados por uno de los dispositivos del editor:

* publisher.beijing.corp.contoso
* publisher.capetown.corp.contoso
* publisher.mumbai.corp.contoso
* publisher.munich0.corp.contoso
* publisher.rio.corp.contoso
* publisher.seattle.corp.contoso

Si ve que ningún dato se envía a IoT Hub, hay un problema con la simulación. Como primer paso de análisis debe analizar los archivos de registro de los componentes de la simulación. Consulte [¿Cómo se pueden obtener datos de registro de los componentes de simulación?](#how-can-i-get-log-data-from-the-simulation-components) Ahora, pruebe a detener e iniciar la simulación, y, si todavía no hay datos enviados, actualice completamente la simulación. Consulte [¿Cómo se actualiza la simulación en la máquina virtual?](#how-do-i-update-the-simulation-in-the-vm)

### <a name="how-do-i-enable-an-interactive-map-in-my-connected-factory-solution"></a>¿Cómo se habilita un mapa interactivo en una solución de factoría conectada?

Para habilitar un mapa interactivo en la solución Connected Factory, debe tener una cuenta de Azure Maps.

Cuando se implementa desde [www.azureiotsolutions.com](https://www.azureiotsolutions.com), el proceso de implementación agrega una cuenta de Azure Maps al grupo de recursos que contiene los servicios del acelerador de soluciones.

Al realizar la implementación mediante el script `build.ps1` del repositorio de GitHub de Connected Factory, establezca la variable de entorno `$env:MapApiQueryKey` de la ventana de compilación como la [clave de la cuenta de Azure Maps](../azure-maps/how-to-manage-account-keys.md). El mapa interactivo se habilita de manera automática.

También puede agregar una clave de cuenta de Azure Maps al acelerador de soluciones después de la implementación. Vaya a Azure Portal y acceda al recurso App Service en la implementación de Factoría conectada. Vaya a **Configuración de la aplicación**, donde encontrará una sección denominada **Configuración de la aplicación**. Establezca **MapApiQueryKey** como la [clave de la cuenta de Azure Maps](../azure-maps/how-to-manage-account-keys.md). Guarde la configuración, vaya a la **información general** y reinicie App Service.

### <a name="how-do-i-create-an-azure-maps-account"></a>¿Cómo se crea una cuenta de Azure Maps?

Consulte [Procedimientos de administración de la cuenta y las claves de Azure Maps](../azure-maps/how-to-manage-account-keys.md).

### <a name="how-to-obtain-your-azure-maps-account-key"></a>Obtención de la clave de cuenta de Azure Maps

Consulte [Procedimientos de administración de la cuenta y las claves de Azure Maps](../azure-maps/how-to-manage-account-keys.md).

### <a name="how-do-enable-the-interactive-map-while-debugging-locally"></a>¿Cómo se habilita el mapa interactivo al depurar localmente?

Para habilitar el mapa interactivo al depurar localmente, establezca el valor del ajuste `MapApiQueryKey` en los archivos `local.user.config` y `<yourdeploymentname>.user.config` en la raíz de la implementación en el valor de la **QueryKey** que ha copiado anteriormente.

### <a name="how-do-i-use-a-different-image-at-the-home-page-of-my-dashboard"></a>¿Cómo puedo usar una imagen distinta en la página principal de mi panel?

Para cambiar la imagen estática que se muestra en la página principal del panel, sustituya la imagen `WebApp\Content\img\world.jpg`. A continuación, recompile y vuelva a implementar WebApp.

### <a name="how-do-i-use-non-opc-ua-devices-with-connected-factory"></a>¿Cómo se puede usar un dispositivo que no sea de OPC UA con la factoría conectada?

Para enviar datos de telemetría desde dispositivos que no son de OPC UA a la factoría conectada:

1. [Configure una nueva estación en la topología de la factoría conectada](iot-accelerators-connected-factory-configure.md) en el archivo `ContosoTopologyDescription.json`.

1. Introduzca los datos de telemetría en un formato JSON compatible en la factoría conectada:

    ```json
    [
      {
        "ApplicationUri": "<the_value_of_OpcUri_of_your_station",
        "DisplayName": "<name_of_the_datapoint>",
        "NodeId": "value_of_NodeId_of_your_datapoint_in_the_station",
        "Value": {
          "Value": <datapoint_value>,
          "SourceTimestamp": "<timestamp>"
        }
      }
    ]
    ```

1. El formato de `<timestamp>` es: `2017-12-08T19:24:51.886753Z`

1. Reinicie la instancia de App Service de factoría conectada.

### <a name="next-steps"></a>Pasos siguientes

También puede explorar algunas de las demás características y funcionalidades de los aceleradores de soluciones de IoT:

* [Implementación del acelerador de soluciones de Connected Factory](quickstart-connected-factory-deploy.md)
* [Seguridad de Internet de las cosas desde el principio](../iot-fundamentals/iot-security-ground-up.md)