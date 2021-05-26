---
title: 'Tutorial: Creación de una jerarquía de dispositivos IoT Edge con Azure IoT Edge'
description: En este tutorial se muestra cómo crear una estructura jerárquica de dispositivos IoT Edge mediante puertas de enlace.
author: v-tcassi
manager: philmea
ms.author: v-tcassi
ms.date: 2/26/2021
ms.topic: tutorial
ms.service: iot-edge
services: iot-edge
monikerRange: '>=iotedge-2020-11'
ms.openlocfilehash: 7362c134ce25341a9ce53659c3a1f7ff2889e7fa
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110083591"
---
# <a name="tutorial-create-a-hierarchy-of-iot-edge-devices"></a>Tutorial: Creación de una jerarquía de dispositivos IoT Edge

[!INCLUDE [iot-edge-version-202011](../../includes/iot-edge-version-202011.md)]

Implemente nodos de Azure IoT Edge en redes organizadas en capas jerárquicas. Cada capa de una jerarquía es un dispositivo de puerta de enlace que controla los mensajes y las solicitudes de los dispositivos de la capa que se encuentra debajo.

Puede estructurar una jerarquía de dispositivos para que solo la capa superior tenga conectividad a la nube y las capas inferiores solo puedan comunicarse con las capas superior e inferior adyacentes. Estas capas de red constituyen la base de la mayoría de las redes industriales que siguen la [norma ISA-95](https://en.wikipedia.org/wiki/ANSI/ISA-95).

El objetivo de este tutorial es crear una jerarquía de dispositivos IoT Edge que simule un entorno de producción simplificado. Al final, implementará el [módulo del sensor de temperatura simulado](https://azuremarketplace.microsoft.com/marketplace/apps/azure-iot.simulated-temperature-sensor) en un dispositivo de nivel inferior sin acceso a Internet mediante la descarga de imágenes de contenedor a través de la jerarquía.

Para lograr este objetivo, este tutorial le guiará a través de la creación de una jerarquía de dispositivos IoT Edge, la implementación de contenedores del entorno de ejecución de Azure IoT Edge en los dispositivos y la configuración local de estos. En este tutorial, usará una herramienta de configuración automatizada para hacer lo siguiente:

> [!div class="checklist"]
>
> * Crear y definir las relaciones en una jerarquía de dispositivos IoT Edge.
> * Configurar el entorno de ejecución de Azure IoT Edge en los dispositivos de la jerarquía.
> * Instalar certificados coherentes en la jerarquía de dispositivos.
> * Agregar cargas de trabajo a los dispositivos de la jerarquía.
> * Use el [módulo de proxy de API de IoT Edge](https://azuremarketplace.microsoft.com/marketplace/apps/azure-iot.azureiotedge-api-proxy?tab=Overview) para enrutar de forma segura el tráfico HTTP por un único puerto desde los dispositivos de nivel inferior.

>[!TIP]
>Este tutorial incluye una combinación de pasos manuales y automatizados para proporcionar una presentación de las características de IoT Edge anidadas.
>
>Si prefiere una visión completamente automatizada de la configuración de una jerarquía de dispositivos IoT Edge, puede seguir el [ejemplo de Azure IoT Edge para IoT industrial](https://aka.ms/iotedge-nested-sample) con scripts. En este escenario con scripts se implementan máquinas virtuales de Azure como dispositivos preconfigurados para simular el entorno de una fábrica.
>
>Si quiere conocer en profundidad los pasos manuales para crear y administrar una jerarquía de dispositivos IoT Edge, consulte [la guía paso a paso sobre las jerarquías de puerta de enlace de dispositivos IoT Edge](how-to-connect-downstream-iot-edge-device.md).

En este tutorial, se definen los siguientes niveles de red:

* **Nivel superior**: Los dispositivos IoT Edge de este nivel se pueden conectar directamente a la nube.

* **Capas inferiores**: los dispositivos IoT Edge en las capas situadas debajo de la superior no se pueden conectar directamente a la nube. Deben pasar por uno o más dispositivos IoT Edge intermediarios para enviar y recibir datos.

En este tutorial se usa una jerarquía de dos dispositivos para simplificar, mostrada a continuación. El **dispositivo de nivel superior** representa un dispositivo en la capa superior de la jerarquía, que se puede conectar directamente a la nube. También se hace referencia a este dispositivo como **dispositivo primario**. El otro dispositivo, el **dispositivo de nivel inferior**, representa un dispositivo en la capa inferior de la jerarquía, que no se puede conectar directamente a la nube. Puede agregar más dispositivos de capa inferior para representar el entorno de producción, según sea necesario. Los dispositivos de las capas inferiores también se denominarán **dispositivos secundarios**.

![Estructura de la jerarquía del tutorial, que contiene dos dispositivos: el de nivel superior y el de nivel inferior](./media/tutorial-nested-iot-edge/tutorial-hierarchy-diagram.png)

## <a name="prerequisites"></a>Requisitos previos

Para crear una jerarquía de dispositivos IoT Edge, necesitará:

* Un equipo (Windows o Linux) con conectividad a Internet.
* Una cuenta de Azure con una suscripción válida. Si no tiene una [suscripción a Azure](../guides/developer/azure-developer-guide.md#understanding-accounts-subscriptions-and-billing), cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.
* Una instancia de [IoT Hub](../iot-hub/iot-hub-create-through-portal.md) de nivel estándar o gratis en Azure.
* Un shell de Bash en Azure Cloud Shell que use la CLI de Azure v2.3.1 con la extensión de Azure IoT v0.10.6 o posterior instalada. Este tutorial usa [Azure Cloud Shell](../cloud-shell/overview.md). Si no está familiarizado con Azure Cloud Shell, [consulte un inicio rápido para más información](./quickstart-linux.md#prerequisites).
  * Para ver las versiones actuales de los módulos y extensiones de la CLI de Azure, ejecute [az version](/cli/azure/reference-index?#az_version).
* Un dispositivo Linux para configurar como dispositivo IoT Edge para cada dispositivo de la jerarquía. En este tutorial se usan dos dispositivos. Si no tiene dispositivos disponibles, puede crear máquinas virtuales de Azure para cada dispositivo de la jerarquía mediante el comando siguiente.

   Reemplace el texto del marcador de posición del siguiente comando y ejecute este dos veces, una por cada máquina virtual. Cada máquina virtual necesita un prefijo DNS único, que también servirá como nombre. El prefijo DNS debe ajustarse a la siguiente expresión regular: `[a-z][a-z0-9-]{1,61}[a-z0-9]`.

   ```azurecli
   az deployment group create \
    --resource-group <REPLACE_WITH_YOUR_RESOURCE_GROUP> \
    --template-uri "https://raw.githubusercontent.com/Azure/iotedge-vm-deploy/1.2.0/edgeDeploy.json" \
    --parameters dnsLabelPrefix='<REPLACE_WITH_UNIQUE_DNS_FOR_VIRTUAL_MACHINE>' \
    --parameters adminUsername='azureuser' \
    --parameters authenticationType='sshPublicKey' \
    --parameters adminPasswordOrKey="$(< ~/.ssh/id_rsa.pub)" \
    --query "properties.outputs.[publicFQDN.value, publicSSH.value]" -o tsv
   ```

   La máquina virtual usa claves SSH para autenticar a los usuarios. Si no está familiarizado con la creación y el uso de claves SSH, puede seguir [las instrucciones relativas a los pares de claves SSH pública y privada para máquinas virtuales Linux en Azure](../virtual-machines/linux/mac-create-ssh-keys.md).

   IoT Edge versión 1.2 viene preinstalado con esta plantilla de ARM, lo que ahorra la necesidad de instalar manualmente los recursos en las máquinas virtuales. Si va a instalar IoT Edge en sus propios dispositivos, consulte [Instalación de Azure IoT Edge para Linux (versión 1.2)](how-to-install-iot-edge.md) o [Actualización de IoT Edge a la versión 1.2.](how-to-update-iot-edge.md#special-case-update-from-10-or-11-to-12)

   Después de crear correctamente una máquina virtual con esta plantilla de ARM, se generará el identificador `SSH` y el nombre de dominio completo de la máquina virtual (`FQDN`). En pasos posteriores usará el identificador SSH y el nombre de dominio completo o la dirección IP de cada máquina virtual para la configuración, así que no debe perder de vista esta información. A continuación, se muestra una salida de ejemplo.

   >[!TIP]
   >También puede encontrar la dirección IP y el nombre de dominio completo en Azure Portal. En el caso de la dirección IP, vaya a la lista de máquinas virtuales y anote la información del campo **Dirección IP pública**. Para encontrar el nombre de dominio completo, vaya a la página de información general de cada máquina virtual y busque el campo **Nombre DNS**.

   ![Después de la creación, la máquina virtual genera un archivo JSON que contiene su identificador SSH.](./media/tutorial-nested-iot-edge/virtual-machine-outputs.png)

* Asegúrese de que los siguientes puertos estén abiertos para todos los dispositivos de entrada, excepto para el dispositivo de capa inferior: 8000, 443, 5671, 8883:
  * 8000: se usa para extraer las imágenes de contenedor de Docker mediante el proxy de API.
  * 443: se usa entre los centros de conectividad perimetrales primario y secundario para las llamadas de API REST.
  * 5671, 8883: se usan para AMQP y MQTT.

  Para más información, consulte [Apertura de puertos en una máquina virtual con Azure Portal](../virtual-machines/windows/nsg-quickstart-portal.md).

## <a name="configure-your-iot-edge-device-hierarchy"></a>Configuración de la jerarquía de dispositivos IoT Edge

### <a name="create-a-hierarchy-of-iot-edge-devices"></a>Creación de una jerarquía de dispositivos IoT Edge

Los dispositivos IoT Edge componen las capas de la jerarquía. En este tutorial se creará una jerarquía de dos dispositivos IoT Edge: el **dispositivo de nivel superior** y su secundario, el **dispositivo de nivel inferior**. Puede crear dispositivos secundarios adicionales según sea necesario.

Para crear y configurar la jerarquía de dispositivos IoT Edge, usará la herramienta `iotedge-config`. Esta herramienta simplifica la configuración de la jerarquía gracias a la automatización y la condensación de varios pasos en dos:

1. Configuración de las opciones de nube y preparación de la configuración de cada dispositivo, lo que incluye:

   * Creación de los dispositivos en un centro de IoT
   * Establecimiento de las relaciones primario-secundario para autorizar la comunicación entre los dispositivos
   * Generación de una cadena de certificados para cada dispositivo a fin de establecer una comunicación segura entre ellos
   * Generación de archivos de configuración para cada dispositivo

1. Instalación de la configuración de cada dispositivo, lo que incluye:

   * Instalación de certificados en cada dispositivo
   * Aplicación de los archivos de configuración a cada dispositivo

La herramienta `iotedge-config` también realizará las implementaciones de los módulos en el dispositivo IoT Edge de forma automática.

Para usar la herramienta `iotedge-config` para crear y configurar la jerarquía, siga los pasos que se indican a continuación en la CLI de Azure:

1. En [Azure Cloud Shell](https://shell.azure.com/), cree un directorio para los recursos del tutorial:

   ```bash
   mkdir nestedIotEdgeTutorial
   ```

1. Descargue la versión de la herramienta de configuración y las plantillas de configuración:

   ```bash
   cd ~/nestedIotEdgeTutorial
   wget -O iotedge_config.tar "https://github.com/Azure-Samples/iotedge_config_cli/releases/download/latest/iotedge_config_cli.tar.gz"
   tar -xvf iotedge_config.tar
   ```

   Al realizar esta acción, se crea la carpeta `iotedge_config_cli_release` en el directorio del tutorial.

   El archivo de plantilla que se usa para crear la jerarquía de dispositivos es `iotedge_config.yaml` que se encuentra en `~/nestedIotEdgeTutorial/iotedge_config_cli_release/templates/tutorial`. En el mismo directorio, `deploymentLowerLayer.json` es un archivo de implementación JSON que contiene instrucciones sobre los módulos que se deben implementar en el **dispositivo de capa inferior**. El archivo `deploymentTopLayer.json` es el mismo, pero para el **dispositivo de capa superior**, ya que los módulos implementados en cada dispositivo no son los mismos. El archivo `device_config.toml` es una plantilla para configuraciones de dispositivos IoT Edge y se usará para generar automáticamente las agrupaciones de configuración de los dispositivos de la jerarquía.

   Si quiere echar un vistazo al código fuente y los scripts de la herramienta `iotedge-config`, consulte [el repositorio de ejemplos de Azure en GitHub](https://github.com/Azure-Samples/iotedge_config_cli).

1. Abra la plantilla de configuración del tutorial y edítela con su información:

   ```bash
   code ~/nestedIotEdgeTutorial/iotedge_config_cli_release/templates/tutorial/iotedge_config.yaml
   ```

   En la sección **iothub**, rellene los campos `iothub_hostname` y `iothub_name` con la información. Esta información se puede encontrar en la página de información general de IoT Hub en Azure Portal.

   En la sección **certificates**, puede rellenar los campos con las rutas de acceso absolutas al certificado y la clave. Si deja estos campos en blanco, el script generará automáticamente certificados de prueba autofirmados para que los use. Si no está familiarizado con el modo en que se usan los certificados en un escenario de puerta de enlace, consulte [la sección sobre los certificados en la guía paso a paso](how-to-connect-downstream-iot-edge-device.md#prepare-certificates).

   En la sección **configuration**, `template_config_path` es la ruta de acceso a la plantilla `device_config.toml` que se usa para crear las configuraciones del dispositivo. El campo `default_edge_agent` determina qué dispositivos de capa inferior de la imagen del agente de Edge se extraerán y de dónde.

   En la sección **edgedevices**, en un escenario de producción, puede editar el árbol de jerarquía para reflejar la estructura deseada. Para los fines de tutorial, acepte el árbol predeterminado. Para cada dispositivo, hay un campo `device_id` en el que puede nombrar a los dispositivos. También está el campo `deployment`, que especifica la ruta de acceso al archivo JSON de implementación de ese dispositivo.

   Además, puede registrar manualmente los dispositivos IoT Edge en su centro de IoT mediante Azure Portal o Azure Cloud Shell. Para saber cómo, consulte [la guía sobre cómo registrar un dispositivo IoT Edge](how-to-register-device.md).

   También puede definir las relaciones primario-secundario manualmente. Para más información, consulte la sección [Creación de una jerarquía de puerta de enlace](how-to-connect-downstream-iot-edge-device.md#create-a-gateway-hierarchy) de la guía paso a paso.

   ![La sección edgedevices del archivo de configuración le permite definir la jerarquía.](./media/tutorial-nested-iot-edge/hierarchy-config-sample.png)

1. Guarde y cierre el archivo:

   `CTRL + S`, `CTRL + Q`

1. Cree un directorio de salida para las agrupaciones de configuración en el directorio de recursos del tutorial:

   ```bash
   mkdir ~/nestedIotEdgeTutorial/iotedge_config_cli_release/outputs
   ```

1. Vaya al directorio `iotedge_config_cli_release` y ejecute la herramienta para crear la jerarquía de dispositivos IoT Edge:

   ```bash
   cd ~/nestedIotEdgeTutorial/iotedge_config_cli_release
   ./iotedge_config --config ~/nestedIotEdgeTutorial/iotedge_config_cli_release/templates/tutorial/iotedge_config.yaml --output ~/nestedIotEdgeTutorial/iotedge_config_cli_release/outputs -f
   ```

   Con la marca `--output`, la herramienta crea los certificados de dispositivo, las agrupaciones de certificados y un archivo de registro en el directorio que prefiera. Con la marca `-f` establecida, la herramienta buscará automáticamente los dispositivos IoT Edge existentes en el centro de IoT y los quitará, para evitar errores y mantener limpio el centro.

   La herramienta de configuración crea los dispositivos IoT Edge y configura las relaciones primario-secundario entre ellos. Opcionalmente, crea certificados para los dispositivos. Si se proporcionan rutas de acceso a los archivos JSON de implementación, la herramienta creará automáticamente estas implementaciones en los dispositivos, pero este paso no es necesario. Por último, la herramienta generará las agrupaciones de configuración para los dispositivos y las colocará en el directorio de salida. Para ver en profundidad los pasos que ha llevado a cabo la herramienta de configuración, consulte el archivo de registro en el directorio de salida.

   ![El script mostrará una topología de la jerarquía tras la ejecución.](./media/tutorial-nested-iot-edge/successful-setup-tool-run.png)

Compruebe bien que la salida de la topología del script sea correcta. Una vez que esté satisfecho con la estructura correcta de la jerarquía, está listo para continuar.

### <a name="configure-the-iot-edge-runtime"></a>Configuración del runtime de IoT Edge

Además del aprovisionamiento de los dispositivos, los pasos de configuración establecen una comunicación de confianza entre los dispositivos de la jerarquía mediante los certificados que ha creado antes. Los pasos también comienzan a establecer la estructura de red de la jerarquía. El dispositivo de nivel superior mantendrá la conectividad a Internet, lo que le permitirá extraer imágenes de su entorno de ejecución de la nube, mientras que los dispositivos de nivel inferior se enrutarán mediante el de nivel superior para acceder a estas imágenes.

Para configurar el entorno de ejecución de IoT Edge, debe aplicar a los dispositivos las agrupaciones de configuración creadas mediante el script de instalación. Las configuraciones varían ligeramente entre el **dispositivo de capa superior** y un **dispositivo de capa inferior**, por lo que debe tener en cuenta el archivo de configuración del dispositivo que aplique a cada uno.

1. Cada dispositivo necesita su agrupación de configuración correspondiente. Puede usar una unidad USB o una [copia de archivo seguro](https://www.ssh.com/ssh/scp/) para trasladar las agrupaciones de configuración a cada dispositivo.

   Asegúrese de enviar la agrupación de configuración correcta a cada dispositivo.

   ```bash
   scp <PATH_TO_CONFIGURATION_BUNDLE> <USER>@<VM_IP_OR_FQDN>:~
   ```

   `:~` significa que la carpeta de configuración se colocará en el directorio principal de la máquina virtual.

1. Inicie sesión en la máquina virtual para aplicar la agrupación de configuración al dispositivo:

   ```bash
   ssh <USER>@<VM_IP_OR_FQDN>
   ```

1. En cada dispositivo, descomprima la agrupación de configuración. Primero tendrá que instalar el archivo comprimido:

   ```bash
   sudo apt install zip
   unzip ~/<PATH_TO_CONFIGURATION_BUNDLE>/<CONFIGURATION_BUNDLE>.zip
   ```

1. En cada dispositivo, aplique la agrupación de configuración al dispositivo:

   ```bash
   sudo ./install.sh
   ```

   En el **dispositivo de capa superior,** , recibirá un aviso para escribir el nombre de host. En el **dispositivo de capa inferior,** , se le pedirá el nombre de host y el nombre de host del dispositivo primario. En cada aviso, proporcione la dirección IP o el nombre de dominio completo adecuados. Puede usar cualquiera de ellos, pero sea coherente en su elección en todos los dispositivos. A continuación, se muestra la salida del script de instalación.

   Si quiere ver más de cerca las modificaciones que se realizan en el archivo de configuración del dispositivo, consulte [la sección sobre cómo configurar IoT Edge en los dispositivos en la guía paso a paso](how-to-connect-downstream-iot-edge-device.md#configure-iot-edge-on-devices).

  ![La instalación de las agrupaciones de configuración actualizará los archivos config.toml en el dispositivo y reiniciará todos los servicios IoT Edge automáticamente.](./media/tutorial-nested-iot-edge/configuration-install-output.png)

Si ha completado correctamente los pasos anteriores, puede comprobar que los dispositivos estén bien configurados.

Ejecute las comprobaciones de configuración y conectividad en los dispositivos. En el **dispositivo de capa superior**:

   ```bash
   sudo iotedge check
   ```

En el **dispositivo de capa inferior**, la imagen de diagnóstico debe pasarse manualmente en el comando:

   ```bash
   sudo iotedge check --diagnostics-image-name <parent_device_fqdn_or_ip>:8000/azureiotedge-diagnostics:1.2
   ```

En el **dispositivo de capa superior**, podrá ver una salida con varias evaluaciones superadas. Es posible que vea algunas advertencias sobre registros, directivas y, en función de la red, directivas de DNS.

<!-- Add pic after GA -->
<!-- KEEP! A sample output of the `iotedge check` is shown below: -->

<!-- KEEP! ![Sample configuration and connectivity results](./media/tutorial-nested-iot-edge/configuration-and-connectivity-check-results.png) -->

Cuando esté satisfecho con las configuraciones correctas en cada dispositivo, ya está listo para continuar.

## <a name="deploy-modules-to-your-devices"></a>Implementación de módulos en los dispositivos

Las implementaciones de módulos en los dispositivos se generaron automáticamente cuando se crearon los dispositivos. La herramienta `iotedge-config-cli` suministró archivos JSON de implementación para los **dispositivos de capa superior e inferior** después de que se crearon. La implementación del módulo estuvo pendiente mientras se configuraba el entorno de ejecución de IoT Edge en cada dispositivo. Una vez configurado el entorno de ejecución, comenzaron las implementaciones en el **dispositivo de capa superior**. Después de finalizar esas implementaciones, el **dispositivo de capa inferior** pudo usar el módulo **IoT Edge API Proxy** (Proxy de API de IoT Edge) para extraer las imágenes necesarias.

En [Azure Cloud Shell](https://shell.azure.com/), puede echar un vistazo al archivo JSON de implementación del **dispositivo de capa superior** para comprender qué módulos se implementaron en el dispositivo:

   ```bash
   cat ~/nestedIotEdgeTutorial/iotedge_config_cli_release/templates/tutorial/deploymentTopLayer.json
   ```

Además de los módulos **IoT Edge Agent** (Agente de IoT Edge) y **IoT Edge Hub** (Concentrador de IoT Edge), el **dispositivo de capa superior** recibe el módulo **Docker Registry** y el módulo **IoT Edge API Proxy** (Proxy de API de IoT Edge).

El módulo **Docker Registry** apunta a una instancia existente de Azure Container Registry. En este caso, `REGISTRY_PROXY_REMOTEURL` apunta a Microsoft Container Registry. En `createOptions`, puede ver que se comunica en el puerto 5000.

El módulo **IoT Edge API Proxy** (Proxy de API de IoT Edge) enruta las solicitudes HTTP a otros módulos, lo que permite que los dispositivos de capa inferior extraigan imágenes del contenedor o inserten blobs en el almacenamiento. En este tutorial, se comunica en el puerto 8000 y está configurado para enviar la ruta de solicitudes de extracción de imágenes de contenedor de Docker al módulo **Docker Registry** en el puerto 5000. Además, cualquier solicitud de carga de Blob Storage se enruta al módulo AzureBlobStorageonIoTEdge en el puerto 11002. Para más información sobre el módulo **IoT Edge API Proxy** (Proxy de API de IoT Edge) y cómo configurarlo, consulte la [guía paso a paso](how-to-configure-api-proxy-module.md) del módulo.

Si quiere ver cómo crear una implementación como esta mediante Azure Portal o Azure Cloud Shell, consulte la [sección sobre los dispositivos de nivel superior en la guía paso a paso](how-to-connect-downstream-iot-edge-device.md#deploy-modules-to-top-layer-devices).

En [Azure Cloud Shell](https://shell.azure.com/), puede echar un vistazo al archivo JSON de implementación del **dispositivo de capa inferior** para comprender qué módulos se implementaron en el dispositivo:

   ```bash
   cat ~/nestedIotEdgeTutorial/iotedge_config_cli_release/templates/tutorial/deploymentLowerLayer.json
   ```

En `systemModules`, puede ver que los módulos del entorno de ejecución del **dispositivo de capa inferior** están establecidos para extraerse de `$upstream:8000`, en lugar de `mcr.microsoft.com`, como hacia el **dispositivo de capa superior**. El **dispositivo de capa inferior** envía solicitudes de imagen de Docker al módulo **IoT Edge API Proxy** (Proxy de API de IoT Edge) en el puerto 8000, ya que no puede extraer directamente las imágenes de la nube. El otro módulo implementado en el **dispositivo de capa inferior**, el módulo **Simulated Temperature Sensor** (Sensor de temperatura simulado), también realiza su solicitud de imágenes a `$upstream:8000`.

Si quiere ver cómo crear una implementación como esta mediante Azure Portal o Azure Cloud Shell, consulte la [sección sobre los dispositivos de capa inferior en la guía paso a paso](how-to-connect-downstream-iot-edge-device.md#deploy-modules-to-lower-layer-devices).

Puede ver el estado de los módulos mediante el comando:

   ```azurecli
   az iot hub module-twin show --device-id <edge_device_id> --module-id '$edgeAgent' --hub-name <iot_hub_name> --query "properties.reported.[systemModules, modules]"
   ```

   Este comando generará todas las propiedades notificadas de edgeAgent. Algunas de estas son útiles para supervisar el estado del dispositivo: *runtime status*, *runtime start time*, *runtime last exit time*, *runtime restart count*.

También puede ver el estado de los módulos en [Azure Portal](https://ms.portal.azure.com/). Vaya a la sección **IoT Edge** del centro de IoT para ver los dispositivos y módulos.

Cuando esté satisfecho con las implementaciones de los módulos, puede continuar.

## <a name="view-generated-data"></a>Visualización de datos generados

El módulo **Simulated Temperature Sensor** que insertó genera datos de entorno de ejemplo. Envía mensajes con la temperatura y la humedad ambiental, la temperatura y la presión de la máquina, y una marca de tiempo.

También puede ver estos mensajes mediante [Azure Cloud Shell](https://shell.azure.com/):

   ```azurecli-interactive
   az iot hub monitor-events -n <iothub_name> -d <lower-layer-device-name>
   ```

## <a name="troubleshooting"></a>Solución de problemas

Ejecute el comando `iotedge check` para comprobar la configuración y solucionar errores.

Puede ejecutar `iotedge check` en una jerarquía anidada, incluso si las máquinas secundarias no tienen acceso directo a Internet.

Cuando se ejecuta `iotedge check` desde la capa inferior, el programa intenta extraer la imagen del dispositivo primario mediante el puerto 443.

En este tutorial, se usa el puerto 8000, por lo que es necesario especificarlo:

```bash
sudo iotedge check --diagnostics-image-name $upstream:8000/azureiotedge-diagnostics:1.2
```

El valor de `azureiotedge-diagnostics` se extrae del registro de contenedor vinculado al módulo del registro. En este tutorial se ha establecido de forma predeterminada en https://mcr.microsoft.com:.

| Nombre | Value |
| - | - |
| `REGISTRY_PROXY_REMOTEURL` | `https://mcr.microsoft.com` |

Si utiliza un registro de contenedor privado, asegúrese de que todas las imágenes (IoTEdgeAPIProxy, edgeAgent, edgeHub, Simulated Temperature Sensor y diagnostics) aparezcan en él.

## <a name="clean-up-resources"></a>Limpieza de recursos

Para evitar gastos, puede eliminar las configuraciones locales y los recursos de Azure que creó en este artículo.

Para eliminar los recursos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y después seleccione **Grupos de recursos**.

2. Seleccione el nombre del grupo de recursos que contiene los recursos de prueba de IoT Edge. 

3. Revise la lista de los recursos contenidos en el grupo de recursos. Si desea eliminar todos ellos, puede seleccionar **Eliminar grupo de recursos**. Si desea eliminar solo algunos de ellos, puede hacer clic en cada recurso para eliminarlos individualmente. 

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, configuró dos dispositivos IoT Edge como puertas de enlace y estableció uno como el dispositivo primario del otro. Después, demostró la extracción de una imagen del contenedor en el dispositivo secundario por medio de una puerta de enlace con el módulo IoT Edge API Proxy (Proxy de API de IoT Edge). Si quiere más información, consulte [la guía paso a paso sobre el uso del módulo de proxy](how-to-configure-api-proxy-module.md).

Para más información sobre el uso de puertas de enlace para crear capas jerárquicas de dispositivos IoT Edge, consulte [la guía paso a paso sobre cómo conectar dispositivos IoT Edge de capa inferior](how-to-connect-downstream-iot-edge-device.md).

Para ver cómo Azure IoT Edge puede crear más soluciones para la empresa, siga con los otros tutoriales.

> [!div class="nextstepaction"]
> [Implementación de un modelo de Azure Machine Learning como módulo](tutorial-deploy-machine-learning.md)