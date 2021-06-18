---
title: 'Instalación y ejecución del contenedor de análisis espacial: Computer Vision'
titleSuffix: Azure Cognitive Services
description: El contenedor de análisis espacial permite detectar personas y distancias.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: conceptual
ms.date: 06/08/2021
ms.author: pafarley
ms.openlocfilehash: 15ef67fb452b4302cc848b88b3dab89d46fb8793
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111746472"
---
# <a name="install-and-run-the-spatial-analysis-container-preview"></a>Instalación y ejecución del contenedor de análisis espacial (versión preliminar)

El contenedor de análisis espacial permite analizar vídeo en streaming en tiempo real para comprender las relaciones espaciales entre las personas, su movimiento y las interacciones con objetos de los entornos físicos. Los contenedores son excelentes para requisitos específicos de control de datos y seguridad.

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services)
* Una vez que tenga la suscripción de Azure, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesComputerVision"  title="cree un recurso de Computer Vision"  target="_blank">creación de un recurso de Computer Vision </a> para el nivel Estándar S1 en Azure Portal con el fin de obtener la clave y el punto de conexión. Una vez que se implemente, haga clic en **Ir al recurso**.
    * Necesitará la clave y el punto de conexión del recurso que se crea para ejecutar el contenedor de análisis espacial. Los usará más adelante.

### <a name="spatial-analysis-container-requirements"></a>Requisitos del contenedor de análisis espacial

Para ejecutar el contenedor de análisis espacial se necesita un dispositivo de proceso con una [GPU NVIDIA Tesla T4](https://www.nvidia.com/en-us/data-center/tesla-t4/). Se recomienda usar [Azure Stack Edge](https://azure.microsoft.com/products/azure-stack/edge/) con aceleración de GPU; sin embargo, el contenedor se ejecuta en cualquier otra máquina de escritorio que cumple con los requisitos mínimos. Haremos referencia a este dispositivo como el equipo host.

#### <a name="azure-stack-edge-device"></a>[Dispositivo Azure Stack Edge](#tab/azure-stack-edge)

Azure Stack Edge es una solución de hardware como servicio y un dispositivo informático perimetral habilitado para inteligencia artificial que cuenta con funcionalidades de transferencia de datos de red. Para instrucciones detalladas sobre la preparación y configuración, consulte la [documentación de Azure Stack Edge](../../databox-online/azure-stack-edge-deploy-prep.md).

#### <a name="desktop-machine"></a>[Máquina de escritorio](#tab/desktop-machine)

#### <a name="minimum-hardware-requirements"></a>Requisitos mínimos de hardware

* 4 GB de RAM del sistema
* 4 GB de RAM de GPU
* CPU de 8 núcleos
* 1 GPU NVIDIA Tesla T4
* 20 GB de espacio en HDD

#### <a name="recommended-hardware"></a>Hardware recomendado

* 32 GB de RAM del sistema
* 16 GB de RAM de GPU
* CPU de 8 núcleos
* 2 GPU NVIDIA Tesla T4
* 50 GB de espacio en SSD

En este artículo, descargará e instalará los paquetes de software siguientes. El equipo host debe poder ejecutar lo siguiente (consulte las instrucciones a continuación):

* [Controladores de gráficos de NVIDIA](https://docs.nvidia.com/datacenter/tesla/tesla-installation-notes/index.html) y [NVIDIA CUDA Toolkit](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html)
* Configuraciones para [NVIDIA MPS](https://docs.nvidia.com/deploy/pdf/CUDA_Multi_Process_Service_Overview.pdf) (servicio multiproceso).
* [Docker CE](https://docs.docker.com/install/linux/docker-ce/ubuntu/#install-docker-engine---community-1) y [NVIDIA-Docker2](https://github.com/NVIDIA/nvidia-docker) 
* Entorno de ejecución de [Azure IoT Edge](../../iot-edge/how-to-install-iot-edge.md).

#### <a name="azure-vm-with-gpu"></a>[Azure VM con GPU](#tab/virtual-machine)
En nuestro ejemplo, usaremos una [máquina virtual de la serie NC](../../virtual-machines/nc-series.md?bc=%2fazure%2fvirtual-machines%2flinux%2fbreadcrumb%2ftoc.json&toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) con una GPU K80.

---

| Requisito | Descripción |
|--|--|
| Cámara | El contenedor de análisis espacial no está asociado a una marca de cámara específica. Los requisitos del dispositivo de cámara son los siguientes: debe admitir el Protocolo de streaming en tiempo real (RTSP) y codificación H.264, debe ser accesible para el equipo host y debe admitir streaming con una resolución de 15 FPS y 1080p. |
| SO Linux | [Ubuntu Desktop 18.04 LTS](http://releases.ubuntu.com/18.04/) debe estar instalado en el equipo host.  |

## <a name="set-up-the-host-computer"></a>Configuración del equipo host

Se recomienda usar un dispositivo Azure Stack Edge para el equipo host. Haga clic en **Máquina de escritorio** si va a configurar otro tipo de dispositivo, o bien en **Máquina virtual** si va a usar una máquina virtual.

#### <a name="azure-stack-edge-device"></a>[Dispositivo Azure Stack Edge](#tab/azure-stack-edge)

### <a name="configure-compute-on-the-azure-stack-edge-portal"></a>Configuración del proceso en el portal de Azure Stack Edge 
 
El análisis espacial usa las características de proceso de Azure Stack Edge para ejecutar una solución de inteligencia artificial. Para habilitar las características de proceso, asegúrese de que se cumple lo siguiente: 

* [Conectó y activó](../../databox-online/azure-stack-edge-deploy-connect-setup-activate.md) el dispositivo Azure Stack Edge. 
* Tiene un sistema cliente de Windows que ejecuta PowerShell 5.0 o posterior para acceder al dispositivo.  
* Para implementar un clúster de Kubernetes, debe configurar el dispositivo Azure Stack Edge mediante la **UI local** en [Azure Portal](https://portal.azure.com/): 
  1. Habilite la característica de proceso en el dispositivo Azure Stack Edge. Para habilitar el proceso, vaya a la página **Proceso** en la interfaz web del dispositivo. 
  2. Seleccione una interfaz de red que quiera habilitar para el proceso y, luego, haga clic en **Habilitar**. Se creará un conmutador virtual en el dispositivo, en esa interfaz de red.
  3. Deje en blanco las direcciones IP de los nodos de prueba de Kubernetes y las de los servicios externos de Kubernetes.
  4. Haga clic en **Aplicar**. Esta operación puede tardar unos dos minutos. 

![Configurar el proceso](media/spatial-analysis/configure-compute.png)

### <a name="set-up-an-edge-compute-role-and-create-an-iot-hub-resource"></a>Configuración de un rol de proceso de Edge y creación de un recurso de IoT Hub

En [Azure Portal](https://portal.azure.com/), vaya al recurso de Azure Stack Edge. En la página **Información general** o en la lista de navegación, haga clic en el botón **Comenzar** del proceso de Edge. En el icono  **Configurar proceso de Edge** , haga clic en **Configurar**. 

![Link](media/spatial-analysis/configure-edge-compute-tile.png)

En la página  **Configurar proceso de Edge** , elija una instancia existente de IoT Hub o elija crear una nueva. De manera predeterminada, se usa un plan de tarifa Estándar (S1) para crear un recurso de IoT Hub. Para usar un recurso de IoT Hub de nivel Gratis, cree uno y selecciónelo. El recurso de IoT Hub usa la misma suscripción y el mismo grupo de recursos que el recurso de Azure Stack Edge. 

Haga clic en **Crear**. La creación del recurso de IoT Hub puede tardar un par de minutos. Después de crear el recurso de IoT Hub, el icono  **Configurar proceso de Edge** se actualizará para mostrar la configuración nueva. Para configurar que se configuró el rol del proceso de Edge, seleccione  **Ver configuración** en el icono  **Configurar proceso** .

Cuando el rol de proceso de Edge está configurado en el dispositivo de Edge, este crea dos dispositivos: uno IoT y el otro IoT Edge. Ambos se pueden ver en el recurso de IoT Hub. El entorno de ejecución de Azure IoT Edge ya estará en ejecución en el dispositivo IoT Edge.

> [!NOTE]
> * Actualmente, solo la plataforma Linux está disponible para los dispositivos IoT Edge. Si necesita ayuda para solucionar problemas con el dispositivo Azure Stack Edge, consulte el artículo sobre [registro y solución de problemas](spatial-analysis-logging.md).
> * Para obtener más información sobre cómo configurar un dispositivo IoT Edge para comunicarse a través de un servidor proxy, consulte [Configuración de un dispositivo IoT Edge para comunicarse a través de un servidor proxy](../../iot-edge/how-to-configure-proxy-support.md#azure-portal).

###  <a name="enable-mps-on-azure-stack-edge"></a>Habilitación de MPS en Azure Stack Edge 

1. Ejecute una sesión de Windows PowerShell como administrador. 

2. Asegúrese de que el servicio Administración remota de Windows se ejecuta en el cliente. En el terminal de PowerShell, use el comando siguiente: 
    
    ```powershell
    winrm quickconfig
    ```
    
    Si ve advertencias sobre una excepción del firewall, revise el tipo de conexión de red y consulte la documentación sobre la [Administración remota de Windows](/windows/win32/winrm/installation-and-configuration-for-windows-remote-management).

3. Asigne una variable a la dirección IP del dispositivo. 
    
    ```powershell
    $ip = "<device-IP-address>" 
    ```
    
4. Para agregar la dirección IP del dispositivo a la lista de hosts de confianza del cliente, use el comando siguiente: 
    
    ```powershell
    Set-Item WSMan:\localhost\Client\TrustedHosts $ip -Concatenate -Force 
    ```

5. Inicie una sesión de Windows PowerShell en el dispositivo. 

    ```powershell
    Enter-PSSession -ComputerName $ip -Credential $ip\EdgeUser -ConfigurationName Minishell 
    ```

6. Proporcione la contraseña cuando se le solicite. Use la misma contraseña que se emplea para iniciar sesión en la interfaz de usuario web local. La contraseña de la interfaz de usuario web local predeterminada es `Password1`.

Escriba `Start-HcsGpuMPS` para iniciar el servicio MPS en el dispositivo. 

Si necesita ayuda para solucionar problemas con el dispositivo Azure Stack Edge, consulte [Solución de problemas del dispositivo Azure Stack Edge](spatial-analysis-logging.md#troubleshooting-the-azure-stack-edge-device). 

#### <a name="desktop-machine"></a>[Máquina de escritorio](#tab/desktop-machine)

Siga estas instrucciones si el equipo host no es un dispositivo Azure Stack Edge.

#### <a name="install-nvidia-cuda-toolkit-and-nvidia-graphics-drivers-on-the-host-computer"></a>Instalación de los controladores de gráficos de NVIDIA y NVIDIA CUDA Toolkit en el equipo host

Use el script de Bash siguiente para instalar los controladores de gráficos de NVIDIA necesarios y CUDA Toolkit.

```bash
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/cuda-ubuntu1804.pin
sudo mv cuda-ubuntu1804.pin /etc/apt/preferences.d/cuda-repository-pin-600
sudo apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/7fa2af80.pub
sudo add-apt-repository "deb http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/ /"
sudo apt-get update
sudo apt-get -y install cuda
```

Reinicie la máquina y ejecute el comando siguiente.

```bash
nvidia-smi
```

Debería ver la siguiente salida.

![Salida del controlador NVIDIA](media/spatial-analysis/nvidia-driver-output.png)

### <a name="install-docker-ce-and-nvidia-docker2-on-the-host-computer"></a>Instalación de Docker CE y nvidia-docker2 en el equipo host

Instale Docker CE en el equipo host.

```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io
```

Instale el paquete de software *nvidia-docker-2*.

```bash
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
sudo apt-get update
sudo apt-get install -y docker-ce nvidia-docker2
sudo systemctl restart docker
```

## <a name="enable-nvidia-mps-on-the-host-computer"></a>Habilitación de NVIDIA MPS en el equipo host

> [!TIP]
> * No instale MP si la capacidad de proceso de la GPU es inferior a 7.x (Volta). Consulte [Compatibilidad de CUDA](https://docs.nvidia.com/deploy/cuda-compatibility/index.html#support-title) como referencia. 
> * Ejecute las instrucciones de MPS desde una ventana del terminal en el equipo host, no dentro de la instancia de contenedor Docker.

Para obtener el mejor rendimiento y uso, configure las GPU del equipo host para [Servicio multiproceso (MPS) de NVIDIA](https://docs.nvidia.com/deploy/pdf/CUDA_Multi_Process_Service_Overview.pdf). Ejecute las instrucciones de MPS desde una ventana del terminal en el equipo host,

```bash
# Set GPU(s) compute mode to EXCLUSIVE_PROCESS
sudo nvidia-smi --compute-mode=EXCLUSIVE_PROCESS

# Cronjob for setting GPU(s) compute mode to EXCLUSIVE_PROCESS on boot
echo "SHELL=/bin/bash" > /tmp/nvidia-mps-cronjob
echo "PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin" >> /tmp/nvidia-mps-cronjob
echo "@reboot   root    nvidia-smi --compute-mode=EXCLUSIVE_PROCESS" >> /tmp/nvidia-mps-cronjob

sudo chown root:root /tmp/nvidia-mps-cronjob
sudo mv /tmp/nvidia-mps-cronjob /etc/cron.d/

# Service entry for automatically starting MPS control daemon
echo "[Unit]" > /tmp/nvidia-mps.service
echo "Description=NVIDIA MPS control service" >> /tmp/nvidia-mps.service
echo "After=cron.service" >> /tmp/nvidia-mps.service
echo "" >> /tmp/nvidia-mps.service
echo "[Service]" >> /tmp/nvidia-mps.service
echo "Restart=on-failure" >> /tmp/nvidia-mps.service
echo "ExecStart=/usr/bin/nvidia-cuda-mps-control -f" >> /tmp/nvidia-mps.service
echo "" >> /tmp/nvidia-mps.service
echo "[Install]" >> /tmp/nvidia-mps.service
echo "WantedBy=multi-user.target" >> /tmp/nvidia-mps.service

sudo chown root:root /tmp/nvidia-mps.service
sudo mv /tmp/nvidia-mps.service /etc/systemd/system/

sudo systemctl --now enable nvidia-mps.service
```

## <a name="configure-azure-iot-edge-on-the-host-computer"></a>Configuración de Azure IoT Edge en el equipo host

Para implementar el contenedor de análisis espacial en el equipo host, cree una instancia de un servicio [Azure IoT Hub](../../iot-hub/iot-hub-create-through-portal.md) con el plan de tarifa Estándar (S1) o Gratis (F0). 

Use la CLI de Azure para crear una instancia de Azure IoT Hub. Reemplace los parámetros cuando sea necesario. También puede crear la instancia de Azure IoT Hub en [Azure Portal](https://portal.azure.com/).

```bash
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```
```bash
sudo az login
```
```bash
sudo az account set --subscription "<name or ID of Azure Subscription>"
```
```bash
sudo az group create --name "<resource-group-name>" --location "<your-region>"
```
Consulte [Regiones admitidas](https://azure.microsoft.com/global-infrastructure/services/?products=cognitive-services) para ver las regiones disponibles.
```bash
sudo az iot hub create --name "<iothub-group-name>" --sku S1 --resource-group "<resource-group-name>"
```
```bash
sudo az iot hub device-identity create --hub-name "<iothub-name>" --device-id "<device-name>" --edge-enabled
```

Necesitará instalar [Azure IoT Edge](../../iot-edge/how-to-install-iot-edge.md), versión 1.0.9. Siga estos pasos para descargar la versión correcta:

Ubuntu Server 18.04:
```bash
curl https://packages.microsoft.com/config/ubuntu/18.04/multiarch/prod.list > ./microsoft-prod.list
```

Copie la lista generada.
```bash
sudo cp ./microsoft-prod.list /etc/apt/sources.list.d/
```

Instale la clave pública de GPG de Microsoft.

```bash
curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg
```
```bash
sudo cp ./microsoft.gpg /etc/apt/trusted.gpg.d/
```

Actualice las listas de paquetes en el dispositivo.

```bash
sudo apt-get update
```

Instale la versión 1.0.9:

```bash
sudo apt-get install iotedge=1.0.9* libiothsm-std=1.0.9*
```

A continuación, registre el equipo host como dispositivo IoT Edge en la instancia de IoT Hub mediante una [cadena de conexión](../../iot-edge/how-to-register-device.md).

Debe conectar el dispositivo IoT Edge a la instancia de Azure IoT Hub. Debe copiar la cadena de conexión del dispositivo IoT Edge que creó anteriormente. También puede ejecutar el comando siguiente en la CLI de Azure.

```bash
sudo az iot hub device-identity connection-string show --device-id my-edge-device --hub-name test-iot-hub-123
```

En el equipo host, abra `/etc/iotedge/config.yaml` para su edición. Reemplace `ADD DEVICE CONNECTION STRING HERE` por la cadena de conexión. Guarde y cierre el archivo. Ejecute este comando para reiniciar el servicio IoT Edge en el equipo host.

```bash
sudo systemctl restart iotedge
```

Implemente el contenedor de análisis espacial como módulo de IoT en el equipo host, ya sea desde [Azure Portal](../../iot-edge/how-to-deploy-modules-portal.md) o desde la [CLI de Azure](../cognitive-services-apis-create-account-cli.md?tabs=windows). Si usa el portal, establezca el URI de imagen en la ubicación de la instancia de Azure Container Registry. 

Use los pasos siguientes para implementar el contenedor mediante la CLI de Azure.

#### <a name="azure-vm-with-gpu"></a>[Azure VM con GPU](#tab/virtual-machine)

También es posible usar una máquina virtual de Azure con una GPU para llevar a cabo un análisis espacial. En el ejemplo siguiente se usará una máquina virtual de la [serie CN](../../virtual-machines/nc-series.md?bc=%2fazure%2fvirtual-machines%2flinux%2fbreadcrumb%2ftoc.json&toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) con una GPU K80.

#### <a name="create-the-vm"></a>Creación de la máquina virtual

Abra el asistente [Crear máquina virtual](https://ms.portal.azure.com/#create/Microsoft.VirtualMachine) desde Azure Portal.

Asigne un nombre a la máquina virtual y seleccione la región (EE. UU.) Oeste de EE. UU. 2. Asegúrese de establecer `Availability Options` en "No se requiere redundancia de la infraestructura". Consulte la figura para ver la configuración completa y el paso siguiente para identificar el tamaño correcto de la máquina virtual. 

:::image type="content" source="media/spatial-analysis/virtual-machine-instance-details.jpg" alt-text="Detalles de configuración de la máquina virtual" lightbox="media/spatial-analysis/virtual-machine-instance-details.jpg":::

Para localizar el tamaño de la máquina virtual, seleccione "See all sizes" ("Ver todos los tamaños") y consulte la lista de "tamaños de máquina virtual de almacenamiento que no sea premium", que se muestra a continuación.

:::image type="content" source="media/spatial-analysis/virtual-machine-sizes.png" alt-text="Tamaños de máquina virtual" lightbox="media/spatial-analysis/virtual-machine-sizes.png":::

A continuación, seleccione **NC6** o **NC6_Promo**.

:::image type="content" source="media/spatial-analysis/promotional-selection.png" alt-text="selección promocional" lightbox="media/spatial-analysis/promotional-selection.png":::

Cree la máquina virtual. Una vez creada, vaya al recurso de máquina virtual en Azure Portal y seleccione `Extensions` en el panel izquierdo. La ventana Extensiones aparecerá con todas las extensiones disponibles. Seleccione `NVIDIA GPU Driver Extension`, haga clic en Crear y finalice el asistente.

Una vez que la extensión se haya aplicado correctamente, vaya a la página principal de la máquina virtual en Azure Portal y haga clic en `Connect`. Se puede acceder a la máquina virtual mediante SSH o RDP. RDP será útil, ya que permite ver la ventana del visualizador (explicada más adelante). Configure el acceso RDP. Para ello, siga [estos pasos](../../virtual-machines/linux/use-remote-desktop.md) y abra una conexión de escritorio remoto a la máquina virtual.

### <a name="verify-graphics-drivers-are-installed"></a>Comprobar que los controladores de gráficos están instalados

Ejecute el siguiente comando para comprobar que los controladores de gráficos se han instalado correctamente. 

```bash
nvidia-smi
```

Debería ver la siguiente salida.

![Salida del controlador NVIDIA](media/spatial-analysis/nvidia-driver-output.png)

### <a name="install-docker-ce-and-nvidia-docker2-on-the-vm"></a>Instalación de Docker CE y nvidia-docker2 en la máquina virtual

Ejecute los siguientes comandos de uno en uno para instalar Docker CE y nvidia-docker2 en la máquina virtual.

Instale Docker CE en el equipo host.

```bash
sudo apt-get update
```
```bash
sudo apt-get install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common
```
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```
```bash
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```
```bash
sudo apt-get update
```
```bash
sudo apt-get install -y docker-ce docker-ce-cli containerd.io
```


Instale el paquete de software *nvidia-docker-2*.

```bash
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
```
```bash
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
```
```bash
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
```
```bash
sudo apt-get update
```
```bash
sudo apt-get install -y docker-ce nvidia-docker2
```
```bash
sudo systemctl restart docker
```

Una vez instalada y configurada la máquina virtual, siga los pasos que se indican a continuación para configurar Azure IoT Edge. 

## <a name="configure-azure-iot-edge-on-the-vm"></a>Configuración de Azure IoT Edge en la máquina virtual

Para implementar el contenedor de análisis espacial en la máquina virtual, cree una instancia de un servicio [Azure IoT Hub](../../iot-hub/iot-hub-create-through-portal.md) con el plan de tarifa Estándar (S1) o Gratis (F0).

Use la CLI de Azure para crear una instancia de Azure IoT Hub. Reemplace los parámetros cuando sea necesario. También puede crear la instancia de Azure IoT Hub en [Azure Portal](https://portal.azure.com/).

```bash
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```
```bash
sudo az login
```
```bash
sudo az account set --subscription "<name or ID of Azure Subscription>"
```
```bash
sudo az group create --name "<resource-group-name>" --location "<your-region>"
```
Consulte [Regiones admitidas](https://azure.microsoft.com/global-infrastructure/services/?products=cognitive-services) para ver las regiones disponibles.
```bash
sudo az iot hub create --name "<iothub-group-name>" --sku S1 --resource-group "<resource-group-name>"
```
```bash
sudo az iot hub device-identity create --hub-name "<iothub-name>" --device-id "<device-name>" --edge-enabled
```

Necesitará instalar [Azure IoT Edge](../../iot-edge/how-to-install-iot-edge.md), versión 1.0.9. Siga estos pasos para descargar la versión correcta:

Ubuntu Server 18.04:
```bash
curl https://packages.microsoft.com/config/ubuntu/18.04/multiarch/prod.list > ./microsoft-prod.list
```

Copie la lista generada.
```bash
sudo cp ./microsoft-prod.list /etc/apt/sources.list.d/
```

Instale la clave pública de GPG de Microsoft.

```bash
curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg
```
```bash
sudo cp ./microsoft.gpg /etc/apt/trusted.gpg.d/
```

Actualice las listas de paquetes en el dispositivo.

```bash
sudo apt-get update
```

Instale la versión 1.0.9:

```bash
sudo apt-get install iotedge=1.0.9* libiothsm-std=1.0.9*
```

A continuación, registre la máquina virtual como dispositivo IoT Edge en la instancia de IoT Hub mediante una [cadena de conexión](../../iot-edge/how-to-register-device.md).

Debe conectar el dispositivo IoT Edge a la instancia de Azure IoT Hub. Debe copiar la cadena de conexión del dispositivo IoT Edge que creó anteriormente. También puede ejecutar el comando siguiente en la CLI de Azure.

```bash
sudo az iot hub device-identity connection-string show --device-id my-edge-device --hub-name test-iot-hub-123
```

En la máquina virtual, abra `/etc/iotedge/config.yaml` para editarlo. Reemplace `ADD DEVICE CONNECTION STRING HERE` por la cadena de conexión. Guarde y cierre el archivo. Ejecute este comando para reiniciar el servicio IoT Edge en la máquina virtual.

```bash
sudo systemctl restart iotedge
```

Implemente el contenedor de análisis espacial como módulo de IoT en la máquina virtual, ya sea desde [Azure Portal](../../iot-edge/how-to-deploy-modules-portal.md) o desde la [CLI de Azure](../cognitive-services-apis-create-account-cli.md?tabs=windows). Si usa el portal, establezca el URI de imagen en la ubicación de la instancia de Azure Container Registry. 

Use los pasos siguientes para implementar el contenedor mediante la CLI de Azure.

---

### <a name="iot-deployment-manifest"></a>Manifiesto de implementación de IoT

Para optimizar la implementación del contenedor en varios equipos host, puede crear un archivo de manifiesto de implementación para especificar las opciones de creación del contenedor y las variables de entorno. Encontrará un ejemplo de un manifiesto de implementación [para Azure Stack Edge](https://go.microsoft.com/fwlink/?linkid=2142179), [otras máquinas de escritorio](https://go.microsoft.com/fwlink/?linkid=2152270) y [Azure VM con GPU](https://go.microsoft.com/fwlink/?linkid=2152189) en GitHub.

En la tabla siguiente se muestran las distintas variables de entorno que usa el módulo IoT Edge. También puede establecerlas en el manifiesto de implementación vinculado anteriormente, mediante el atributo `env` en `spatialanalysis`:

| Nombre de la opción de configuración | Valor | Descripción|
|---------|---------|---------|
| ARCHON_LOG_LEVEL | Info; Verbose (Información; Detallado) | Nivel de registro, seleccione uno de los dos valores|
| ARCHON_SHARED_BUFFER_LIMIT | 377487360 | Sin modificar|
| ARCHON_PERF_MARKER| false| Establézcalo en true para el registro del rendimiento; de lo contrario, debe ser false.| 
| ARCHON_NODES_LOG_LEVEL | Info; Verbose (Información; Detallado) | Nivel de registro, seleccione uno de los dos valores|
| OMP_WAIT_POLICY | PASSIVE | Sin modificar|
| QT_X11_NO_MITSHM | 1 | Sin modificar|
| APIKEY | Su clave de API| Recopile este valor en Azure Portal del recurso de Computer Vision. Puede encontrarlo en la sección **Clave y punto de conexión** del recurso. |
| FACTURACIÓN | Su URI de punto de conexión| Recopile este valor en Azure Portal del recurso de Computer Vision. Puede encontrarlo en la sección **Clave y punto de conexión** del recurso.|
| EULA | accept | Este valor se debe establecer en *accept* para que el contenedor se ejecute |
| PANTALLA | :1 | Este valor debe ser igual que la salida de `echo $DISPLAY` en el equipo host. Los dispositivos Azure Stack Edge no tienen pantalla. Esta configuración no es aplicable|
| ARCHON_GRAPH_READY_TIMEOUT | 600 | Agregue esta variable de entorno si la GPU **no** es T4 o NVIDIA 2080 Ti|
| ORT_TENSORRT_ENGINE_CACHE_ENABLE | 0 | Agregue esta variable de entorno si la GPU **no** es T4 o NVIDIA 2080 Ti|
| KEY_ENV | Clave de cifrado ASE | Agregue esta variable de entorno si Video_URL es una cadena ofuscada |
| IV_ENV | Vector de inicialización | Agregue esta variable de entorno si Video_URL es una cadena ofuscada|

> [!IMPORTANT]
> Para poder ejecutar el contenedor, las opciones `Eula`, `Billing` y `ApiKey` deben estar especificadas; de lo contrario, el contenedor no se iniciará.  Para obtener más información, vea [Facturación](#billing).

Una vez que actualice el manifiesto de implementación para los [dispositivos Azure Stack Edge](https://go.microsoft.com/fwlink/?linkid=2142179), la [máquina de escritorio](https://go.microsoft.com/fwlink/?linkid=2152270) o [Azure VM con GPU](https://go.microsoft.com/fwlink/?linkid=2152189) con su propia configuración y selección de operaciones, use el comando siguiente de la [CLI de Azure](../cognitive-services-apis-create-account-cli.md?tabs=windows) para implementar el contenedor en el equipo host, como un módulo IoT Edge.

```azurecli
sudo az login
sudo az extension add --name azure-iot
sudo az iot edge set-modules --hub-name "<iothub-name>" --device-id "<device-name>" --content DeploymentManifest.json --subscription "<name or ID of Azure Subscription>"
```

|Parámetro  |Descripción  |
|---------|---------|
| `--hub-name` | Nombre de la instancia de Azure IoT Hub. |
| `--content` | Nombre del archivo de implementación. |
| `--target-condition` | Nombre del dispositivo IoT Edge para el equipo host. |
| `-–subscription` | Nombre o id. de la suscripción. |

Este comando iniciará la implementación. Vaya a la página de la instancia de Azure IoT Hub en Azure Portal para ver el estado de la implementación. El estado se puede mostrar como *417: La configuración de la implementación del dispositivo no se ha establecido* hasta que el dispositivo termine de descargar las imágenes de contenedor y empiece a ejecutarse.

## <a name="validate-that-the-deployment-is-successful"></a>Comprobación de una implementación correcta

Hay varias maneras de comprobar que el contenedor está en ejecución. Localice *Estado en tiempo de ejecución* en **Configuración de módulo IoT Edge** del módulo de análisis espacial en la instancia de Azure IoT Hub en Azure Portal. Compruebe que el **valor deseado** y el **valor notificado** del *estado en tiempo de ejecución* sea *En ejecución*.

![Comprobación de implementación de ejemplo](./media/spatial-analysis/deployment-verification.png)

Una vez que se complete la implementación y que el contenedor esté en ejecución, el **equipo host** empezará a enviar eventos a Azure IoT Hub. Si usó la versión `.debug` de las operaciones, verá una ventana del visualizador para cada cámara que configuró en el manifiesto de implementación. Ahora puede definir las líneas y zonas que quiere supervisar en el manifiesto de implementación y siga las instrucciones para volver a realizar la implementación. 

## <a name="configure-the-operations-performed-by-spatial-analysis"></a>Configuración de las operaciones realizadas por el análisis espacial

Tendrá que usar [operaciones de análisis espacial](spatial-analysis-operations.md) para configurar el contenedor para usar las cámaras conectadas, configurar las operaciones, etc. Para cada dispositivo de cámara que configure, las operaciones para el análisis espacial generarán un flujo de salida de mensajes JSON enviados a la instancia de Azure IoT Hub.

## <a name="use-the-output-generated-by-the-container"></a>Uso de la salida generada por el contenedor

Si quiere empezar a usar la salida generada por el contenedor, consulte los artículos siguientes:

*    Use el SDK del Centro de eventos de Azure del lenguaje de programación elegido para conectarse al punto de conexión de Azure IoT Hub y recibir los eventos. Para más información, consulte [Leer mensajes del dispositivo a la nube desde el punto de conexión integrado](../../iot-hub/iot-hub-devguide-messages-read-builtin.md). 
*    Configure el enrutamiento de mensajes en la instancia de Azure IoT Hub para enviar los eventos a otros puntos de conexión o guardar los eventos en Azure Blob Storage, etc. Para más información, consulte [Enrutamiento de mensajes de IoT Hub](../../iot-hub/iot-hub-devguide-messages-d2c.md). 

## <a name="running-spatial-analysis-with-a-recorded-video-file"></a>Ejecución de un análisis espacial con un archivo de vídeo grabado

Puede usar el análisis espacial tanto con vídeo grabado como con vídeo en directo. Para usar el análisis espacial para vídeo grabado, intente grabar un archivo de vídeo y guardarlo como archivo mp4. Cree una cuenta de Blob Storage en Azure o use una existente. Luego, actualice la configuración de Blob Storage siguiente en Azure Portal:
    1. Cambie **Se requiere transferencia segura** a **Deshabilitado**.
    2. Cambie **Permitir acceso público a blobs** a **Habilitado**.

Vaya a la sección **Contenedor** y cree un contenedor nuevo o use uno existente. Luego, cargue el archivo de vídeo en el contenedor. Expanda la configuración del archivo cargado y seleccione **Generar SAS**. Asegúrese de establecer una **Fecha de expiración** que abarque el período de prueba. Establezca los **Protocolos permitidos** en *HTTP* (no se admite *HTTPS*).

Haga clic en **Generar URL y token de SAS** y copie la dirección URL de SAS de blob. Reemplace el `https` inicial por `http` y pruebe la dirección URL en un explorador que admita la reproducción de vídeo.

Reemplace `VIDEO_URL` en el manifiesto de implementación para el [dispositivo de Azure Stack Edge](https://go.microsoft.com/fwlink/?linkid=2142179), la [máquina de escritorio](https://go.microsoft.com/fwlink/?linkid=2152270) o [Azure VM con GPU](https://go.microsoft.com/fwlink/?linkid=2152189) por la dirección URL que ha creado para todos los gráficos. Establezca `VIDEO_IS_LIVE` en `false` y vuelva a implementar el contenedor de análisis espacial con el manifiesto actualizado. Observe el ejemplo siguiente.

El módulo de análisis espacial empezará a consumir el archivo de vídeo y también se reproducirá automáticamente de manera continua.


```json
"zonecrossing": {
    "operationId" : "cognitiveservices.vision.spatialanalysis-personcrossingpolygon",
    "version": 1,
    "enabled": true,
    "parameters": {
        "VIDEO_URL": "Replace http url here",
        "VIDEO_SOURCE_ID": "personcountgraph",
        "VIDEO_IS_LIVE": false,
      "VIDEO_DECODE_GPU_INDEX": 0,
        "DETECTOR_NODE_CONFIG": "{ \"gpu_index\": 0, \"do_calibration\": true }",
        "SPACEANALYTICS_CONFIG": "{\"zones\":[{\"name\":\"queue\",\"polygon\":[[0.3,0.3],[0.3,0.9],[0.6,0.9],[0.6,0.3],[0.3,0.3]], \"events\": [{\"type\": \"zonecrossing\", \"config\": {\"threshold\": 16.0, \"focus\": \"footprint\"}}]}]}"
    }
   },

```

## <a name="troubleshooting"></a>Solución de problemas

Si tiene problemas al iniciar o ejecutar el contenedor, siga los pasos para solucionar problemas comunes que aparecen en el artículo [Telemetría y solución de problemas](spatial-analysis-logging.md). En este artículo también se incluye información sobre cómo generar y recopilar registros y sobre la recopilación del estado del sistema.

## <a name="billing"></a>Facturación

El contenedor de análisis espacial envía información de facturación a Azure mediante el uso de un recurso de Computer Vision en la cuenta de Azure. Actualmente, el uso del análisis espacial en la versión preliminar pública es gratis. 

Los contenedores de Azure Cognitive Services no tienen licencia para ejecutarse si no están conectados al punto de conexión de facturación o a las mediciones. Debe habilitar los contenedores para que comuniquen la información de facturación al punto de conexión de facturación en todo momento. Los contenedores de Cognitive Services no envían datos de los clientes (por ejemplo, el vídeo o la imagen que se está analizando) a Microsoft.


## <a name="summary"></a>Resumen

En este artículo, aprendió los conceptos y el flujo de trabajo para la descarga, instalación y ejecución del contenedor de análisis espacial. En resumen:

* El análisis espacial es un contenedor de Linux para Docker.
* Las imágenes del contenedor se descargan desde Microsoft Container Registry.
* Las imágenes de contenedor se ejecutan como módulos de IoT en Azure IoT Edge.
* Configuración del contenedor y su implementación en una máquina host.

## <a name="next-steps"></a>Pasos siguientes

* [Implementación de una aplicación web de recuento de personas](spatial-analysis-web-app.md)
* [Configuración de las operaciones de análisis espacial](spatial-analysis-operations.md)
* [Registro y solución de problemas](spatial-analysis-logging.md)
* [Guía de selección de ubicación de la cámara](spatial-analysis-camera-placement.md)
* [Guía de selección de ubicación de zonas y líneas](spatial-analysis-zone-line-placement.md)
