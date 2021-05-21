---
title: Conexión a un dispositivo GPU de Microsoft Azure Stack Edge Pro y su administración mediante la interfaz de Windows PowerShell | Microsoft Docs
description: Se describe cómo conectarse a una GPU de Azure Stack Edge Pro y cómo administrarla posteriormente mediante la interfaz de Windows PowerShell.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 04/15/2021
ms.author: alkohli
ms.openlocfilehash: ca3a08cedacce23f9b3086a4bd6d8da98042d7c4
ms.sourcegitcommit: eda26a142f1d3b5a9253176e16b5cbaefe3e31b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109737261"
---
# <a name="manage-an-azure-stack-edge-pro-gpu-device-via-windows-powershell"></a>Administración de un dispositivo Azure Stack Edge Pro con GPU mediante Windows PowerShell

[!INCLUDE [applies-to-GPU-and-pro-r-and-mini-r-skus](../../includes/azure-stack-edge-applies-to-gpu-pro-r-mini-r-sku.md)]

La solución Azure Stack Edge Pro GPU permite procesar datos y enviarlos a través de la red a Azure. En este artículo se describen algunas de las tareas de configuración y administración del dispositivo Azure Stack Edge Pro GPU. Puede usar Azure Portal, la interfaz de usuario web local o la interfaz de Windows PowerShell para administrar su dispositivo.

Este artículo se centra en cómo puede conectarse a la interfaz de PowerShell del dispositivo y las tareas que puede realizar con esta interfaz. 


## <a name="connect-to-the-powershell-interface"></a>Conectarse a la interfaz de PowerShell

[!INCLUDE [Connect to admin runspace](../../includes/azure-stack-edge-gateway-connect-minishell.md)]

## <a name="create-a-support-package"></a>Crear un paquete de soporte

[!INCLUDE [Create a support package](../../includes/data-box-edge-gateway-create-support-package.md)]


## <a name="view-device-information"></a>Ver la información del dispositivo
 
[!INCLUDE [View device information](../../includes/data-box-edge-gateway-view-device-info.md)]

## <a name="view-gpu-driver-information"></a>Visualización de la información del controlador de GPU

Si el rol de proceso está configurado en su dispositivo, también puede obtener la información del controlador de GPU mediante la interfaz de PowerShell. 

1. [Conéctese a la interfaz de PowerShell](#connect-to-the-powershell-interface).
2. Use `Get-HcsGpuNvidiaSmi` para obtener la información del controlador de GPU del dispositivo.

    En el ejemplo siguiente se muestra el uso de este cmdlet:

    ```powershell
    Get-HcsGpuNvidiaSmi
    ```
    Anote la información del controlador que aparece en la salida de ejemplo de este cmdlet.

    ```powershell    
    +-----------------------------------------------------------------------------+    
    | NVIDIA-SMI 440.64.00    Driver Version: 440.64.00    CUDA Version: 10.2     |    
    |-------------------------------+----------------------+----------------------+    
    | GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |    
    | Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |    
    |===============================+======================+======================|    
    |   0  Tesla T4            On   | 000029CE:00:00.0 Off |                    0 |    
    | N/A   60C    P0    29W /  70W |   1539MiB / 15109MiB |      0%      Default |    
    +-------------------------------+----------------------+----------------------+    
    |   1  Tesla T4           On  | 0000AD50:00:00.0 Off |                    0 |
    | N/A   58C    P0    29W /  70W |    330MiB / 15109MiB |      0%      Default |
    +-------------------------------+----------------------+----------------------+
    ```

## <a name="enable-multi-process-service-mps"></a>Habilitación servicio multiproceso (MPS)

Un servicio multiproceso (MPS) en las GPU de Nvidia proporciona un mecanismo donde varios trabajos pueden compartir las GPU, donde cada trabajo tiene asignado cierto porcentaje de los recursos de la GPU. MPS es una característica en vista previa (GB) en el dispositivo Azure Stack Edge Pro con GPU. Para habilitar MPS en el dispositivo, siga estos pasos:

[!INCLUDE [Enable MPS](../../includes/azure-stack-edge-gateway-enable-mps.md)]

> [!NOTE]
> Cuando se actualizan el software del dispositivo y el clúster de Kubernetes, la configuración de MPS para las cargas de trabajo no se conserva. Tendrá que volver a habilitar MPS.

<!--## Enable compute on private network

Use the `Add-HcsNetRoute` cmdlet to enable compute on a private network. This cmdlet lets you add custom routes on Kubernetes master and worker VMs. 
#### Add new route configuration

IP routing is the process of forwarding a packet based on the destination IP address. For the Kubernetes VMs on your device, you can route the traffic by adding a new route configuration.  

A route configuration is a routing table entry that includes the following fields:

| Parameter | Description  |
|---------|---------|
|Destination     | Either an IP address or an IP address prefix.         |
|Prefix length     | The prefix length corresponding to the address or range of addresses in the destination.        |
|Next hop     | The IP address to which the packet is forwarded.        |
|Interface     | The network interface that forwards the IP packet.        |
|Metric     |Routing metric determines the preferred network interface used to reach the destination.          |


Consider the following information before you add these routes:

- The Kubernetes network where you are adding this route is in a private network and not connected to the internet.
- The device port on which the compute is enabled does not have a gateway configured.
- If you have a flat subnet, then you don't need to add these routes to the private network. You can (optionally) add these routes when there are multiple subnets on your private network.
- You can add these routes only to the Kubernetes master and worker VMs and not to the device (Windows host). 
- The Kubernetes compute need not be configured before you add this route. You can also add or update routes after the Kubernetes compute is configured. You can only add a new route configuration via the PowerShell interface of the device and not through the local UI.
- Make sure that the network interface that you'll use has a static configuration. 

Consider an example where Port 1 and Port 2 on your device are connected to the internet. Ports 3 to Port 6 are on a private network and is the same network that has the Kubernetes master and worker VMs. None of the ports 3 to 6 have a default gateway configured. There are cameras that are connected to the private network. And the camera feed creates a traffic that flows between the camera and the network interfaces on the Kubernetes VMs. 

To add a new custom route, use the cmdlet as follows:

```powershell
Add-HcsNetRoute -InterfaceAlias "Port3" -DestinationPrefix "192.168.21.0/24" -NextHop "192.168.20.1" -RouteMetric 100 
```

Here the compute is enabled on the Port 3 network interface on your device and a virtual switch is created. The above route defines a destination subnet 192.168.21.0/24 and specifies the next hop as 192.168.20.1. This routing configuration has a routing metric of 100. Lower the routing metric, higher the priority assigned to the route.
 

#### Check route configuration for an interface 

Use this cmdlet to check for all the custom route configurations that you added on your device. These routes do not include all the system routes or default routes that already exist on the device. 

```powershell
Get-HcsNetRoute -InterfaceAlias Port3
```


#### Remove a route configuration

Use this cmdlet to remove a route configuration that you added on your device.

```powershell
Remove-HcsNetRoute -InterfaceAlias "Port3" -DestinationPrefix "192.168.21.0/24"
```
-->

## <a name="reset-your-device"></a>Restablecer el dispositivo

[!INCLUDE [Reset your device](../../includes/data-box-edge-gateway-deactivate-device.md)]

## <a name="get-compute-logs"></a>Obtener los registros de proceso

Si el rol de proceso está configurado en su dispositivo, también puede obtener los registros del proceso gracias a la interfaz de PowerShell.

1. [Conéctese a la interfaz de PowerShell](#connect-to-the-powershell-interface).
2. Use `Get-AzureDataBoxEdgeComputeRoleLogs` para obtener los registros del proceso del dispositivo.

    En el ejemplo siguiente se muestra el uso de este cmdlet:

    ```powershell
    Get-AzureDataBoxEdgeComputeRoleLogs -Path "\\hcsfs\logs\myacct" -Credential "username" -FullLogCollection    
    ```

    Aquí tiene hay una descripción de los parámetros que se usan en el cmdlet:
    - `Path`: proporcione una ruta de red al recurso compartido en el que quiere crear el paquete de registros del proceso.
    - `Credential`: proporcione el nombre de usuario del recurso compartido de red. Cuando ejecute este cmdlet, deberá proporcionar la contraseña del recurso compartido.
    - `FullLogCollection`: este parámetro garantiza que el paquete de registros contendrá todos los registros del proceso. De forma predeterminada, el paquete de registros contiene solo un subconjunto de registros.


## <a name="change-kubernetes-pod-and-service-subnets"></a>Cambio de las subredes de pods y de servicio de Kubernetes

De forma predeterminada, Kubernetes en el dispositivo Azure Stack Edge usa las subredes 172.27.0.0/16 y 172.28.0.0/16 para pods y para el servicio respectivamente. Si estas subredes ya están en uso en la red, puede ejecutar el cmdlet `Set-HcsKubeClusterNetworkInfo` para cambiarlas.

Se debe realizar este cambio antes de configurar el proceso en Azure Portal, ya que el clúster de Kubernetes se crea en este paso.

1. [Conexión a la interfaz de PowerShell del dispositivo](#connect-to-the-powershell-interface).
1. Desde dicha interfaz de PowerShell, ejecute lo siguiente:

    `Set-HcsKubeClusterNetworkInfo -PodSubnet <subnet details> -ServiceSubnet <subnet details>`

    Reemplace <subnet details> por el intervalo de subred que quiera usar. 

1. Una vez que haya ejecutado este comando, puede usar el comando `Get-HcsKubeClusterNetworkInfo` para comprobar que las subredes de pods y de servicio han cambiado.

A continuación se muestra una salida de ejemplo de este comando.

```powershell
[10.100.10.10]: PS>Set-HcsKubeClusterNetworkInfo -PodSubnet 10.96.0.1/16 -ServiceSubnet 10.97.0.1/16
[10.100.10.10]: PS>Get-HcsKubeClusterNetworkInfo

Id                                   PodSubnet    ServiceSubnet
--                                   ---------    -------------
6dbf23c3-f146-4d57-bdfc-76cad714cfd1 10.96.0.1/16 10.97.0.1/16
[10.100.10.10]: PS>
```

## <a name="debug-kubernetes-issues-related-to-iot-edge"></a>Depurar problemas de Kubernetes relacionados con IoT Edge

Antes de empezar, debe disponer de lo siguiente:

- Red de proceso configurada. Vea [Tutorial: Configuración de la red para Azure Stack Edge Pro con GPU](azure-stack-edge-gpu-deploy-configure-network-compute-web-proxy.md).
- Rol de proceso configurado en el dispositivo.
    
En un dispositivo Azure Stack Edge Pro GPU que tenga configurado el rol de proceso, puede solucionar problemas o supervisar el dispositivo con dos conjuntos diferentes de comandos.

- Uso de comandos de `iotedge` Estos comandos están disponibles para las operaciones básicas del dispositivo.
- Uso de comandos de `kubectl` Estos comandos están disponibles para un amplio conjunto de operaciones para el dispositivo.

Para ejecutar cualquiera de los comandos anteriores, debe [conectarse a la interfaz de PowerShell](#connect-to-the-powershell-interface).

### <a name="use-iotedge-commands"></a>Usar comandos `iotedge`

Para ver una lista de comandos disponibles, [conéctese a la interfaz de PowerShell](#connect-to-the-powershell-interface) y utilice la función `iotedge`.

```powershell
[10.100.10.10]: PS>iotedge -?                                                                                                                           
Usage: iotedge COMMAND

Commands:
   list
   logs
   restart

[10.100.10.10]: PS>
```

En la tabla siguiente se proporciona una breve descripción de los comandos disponibles para `iotedge`:

|command  |Descripción |
|---------|---------|
|`list`     | Enumere los módulos         |
|`logs`     | Captura los registros de un módulo.        |
|`restart`     | Detiene y reinicia un módulo.         |

#### <a name="list-all-iot-edge-modules"></a>Enumeración de todos los módulos de IoT Edge

Para enumerar todos los módulos que se ejecutan en el dispositivo, use el comando `iotedge list`.

Esta es una salida de ejemplo del comando. Este comando muestra todos los módulos, la configuración asociada y las direcciones IP externas asociadas a los módulos. Por ejemplo, puede acceder a la aplicación del **servidor web** en `https://10.128.44.244`. 


```powershell
[10.100.10.10]: PS>iotedge list

NAME                   STATUS  DESCRIPTION CONFIG                                             EXTERNAL-IP
----                   ------  ----------- ------                                             -----
gettingstartedwithgpus Running Up 10 days  mcr.microsoft.com/intelligentedge/solutions:latest
iotedged               Running Up 10 days  azureiotedge/azureiotedge-iotedged:0.1.0-beta10    <none>
edgehub                Running Up 10 days  mcr.microsoft.com/azureiotedge-hub:1.0             10.128.44.243
edgeagent              Running Up 10 days  azureiotedge/azureiotedge-agent:0.1.0-beta10
webserverapp           Running Up 10 days  nginx:stable                                       10.128.44.244

[10.100.10.10]: PS>
```
#### <a name="restart-modules"></a>Reinicio de módulos

Puede usar el comando `list` para enumerar todos los módulos que se ejecutan en el dispositivo. A continuación, identifique el nombre del módulo que desea reiniciar y úselo con el comando `restart`.

A continuación, se muestra una salida de ejemplo de cómo reiniciar un módulo. En función de la descripción de cuánto tiempo se ejecuta el módulo, puede ver que `cuda-sample1` se ha reiniciado.

```powershell
[10.100.10.10]: PS>iotedge list

NAME         STATUS  DESCRIPTION CONFIG                                          EXTERNAL-IP PORT(S)
----         ------  ----------- ------                                          ----------- -------
edgehub      Running Up 5 days   mcr.microsoft.com/azureiotedge-hub:1.0          10.57.48.62 443:31457/TCP,5671:308
                                                                                             81/TCP,8883:31753/TCP
iotedged     Running Up 7 days   azureiotedge/azureiotedge-iotedged:0.1.0-beta13 <none>      35000/TCP,35001/TCP
cuda-sample2 Running Up 1 days   nvidia/samples:nbody
edgeagent    Running Up 7 days   azureiotedge/azureiotedge-agent:0.1.0-beta13
cuda-sample1 Running Up 1 days   nvidia/samples:nbody

[10.100.10.10]: PS>iotedge restart cuda-sample1
[10.100.10.10]: PS>iotedge list

NAME         STATUS  DESCRIPTION  CONFIG                                          EXTERNAL-IP PORT(S)
----         ------  -----------  ------                                          ----------- -------
edgehub      Running Up 5 days    mcr.microsoft.com/azureiotedge-hub:1.0          10.57.48.62 443:31457/TCP,5671:30
                                                                                              881/TCP,8883:31753/TC
                                                                                              P
iotedged     Running Up 7 days    azureiotedge/azureiotedge-iotedged:0.1.0-beta13 <none>      35000/TCP,35001/TCP
cuda-sample2 Running Up 1 days    nvidia/samples:nbody
edgeagent    Running Up 7 days    azureiotedge/azureiotedge-agent:0.1.0-beta13
cuda-sample1 Running Up 4 minutes nvidia/samples:nbody

[10.100.10.10]: PS>

```

#### <a name="get-module-logs"></a>Obtención de los registros del módulo

Use el comando `logs` para obtener los registros de cualquier módulo de IoT Edge que se ejecute en el dispositivo. 

Si se ha producido un error al crear la imagen de contenedor o durante la extracción de la imagen, ejecute `logs edgeagent`. `edgeagent` es el contenedor de tiempo de ejecución de IoT Edge, que es responsable del aprovisionamiento de otros contenedores. Dado que `logs edgeagent` hace un volcado de todos los registros, una buena forma de ver los errores recientes consiste en usar la opción `--tail `0`. 

Esta es una salida de ejemplo:

```powershell
[10.100.10.10]: PS>iotedge logs cuda-sample2 --tail 10
[10.100.10.10]: PS>iotedge logs edgeagent --tail 10
<6> 2021-02-25 00:52:54.828 +00:00 [INF] - Executing command: "Report EdgeDeployment status: [Success]"
<6> 2021-02-25 00:52:54.829 +00:00 [INF] - Plan execution ended for deployment 11
<6> 2021-02-25 00:53:00.191 +00:00 [INF] - Plan execution started for deployment 11
<6> 2021-02-25 00:53:00.191 +00:00 [INF] - Executing command: "Create an EdgeDeployment with modules: [cuda-sample2, edgeAgent, edgeHub, cuda-sample1]"
<6> 2021-02-25 00:53:00.212 +00:00 [INF] - Executing command: "Report EdgeDeployment status: [Success]"
<6> 2021-02-25 00:53:00.212 +00:00 [INF] - Plan execution ended for deployment 11
<6> 2021-02-25 00:53:05.319 +00:00 [INF] - Plan execution started for deployment 11
<6> 2021-02-25 00:53:05.319 +00:00 [INF] - Executing command: "Create an EdgeDeployment with modules: [cuda-sample2, edgeAgent, edgeHub, cuda-sample1]"
<6> 2021-02-25 00:53:05.412 +00:00 [INF] - Executing command: "Report EdgeDeployment status: [Success]"
<6> 2021-02-25 00:53:05.412 +00:00 [INF] - Plan execution ended for deployment 11
[10.100.10.10]: PS>
```

### <a name="use-kubectl-commands"></a>Uso de comandos de kubectl

En un dispositivo de Azure Stack Edge Pro GPU con el rol de proceso configurado, están disponibles todos los comandos de `kubectl` para supervisar o solucionar problemas de los módulos. Para ver una lista de los comandos disponibles, ejecute `kubectl --help` desde la ventana de comandos.

```PowerShell
C:\Users\myuser>kubectl --help

kubectl controls the Kubernetes cluster manager.

Find more information at: https://kubernetes.io/docs/reference/kubectl/overview/

Basic Commands (Beginner):
    create         Create a resource from a file or from stdin.
    expose         Take a replication controller, service, deployment or pod and expose it as a new Kubernetes Service
    run            Run a particular image on the cluster
    set            Set specific features on objects
    run-container  Run a particular image on the cluster. This command is deprecated, use "run" instead
==============CUT=============CUT============CUT========================

Usage:
    kubectl [flags] [options]

Use "kubectl <command> --help" for more information about a given command.
Use "kubectl options" for a list of global command-line options (applies to all commands).

C:\Users\myuser>
```

Si quiere obtener una lista completa de los comandos de `kubectl`, vaya a la [hoja de referencia de `kubectl`](https://kubernetes.io/docs/reference/kubectl/cheatsheet/).


#### <a name="to-get-ip-of-service-or-module-exposed-outside-of-kubernetes-cluster"></a>Para obtener la dirección IP del servicio o módulo expuesto fuera del clúster de Kubernetes

Para obtener la dirección IP de un servicio de equilibrio de carga o los módulos expuestos fuera del Kubernetes, ejecute el siguiente comando:

`kubectl get svc -n iotedge`

A continuación se muestra una salida de ejemplo de todos los servicios o módulos que se exponen fuera del clúster de Kubernetes. 


```powershell
[10.100.10.10]: PS>kubectl get svc -n iotedge
NAME           TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)                                       AGE
edgehub        LoadBalancer   10.103.52.225   10.128.44.243   443:31987/TCP,5671:32336/TCP,8883:30618/TCP   34h
iotedged       ClusterIP      10.107.236.20   <none>          35000/TCP,35001/TCP                           3d8h
webserverapp   LoadBalancer   10.105.186.35   10.128.44.244   8080:30976/TCP                                16h

[10.100.10.10]: PS>
```
La dirección IP de la columna IP externa corresponde al punto de conexión externo del servicio o del módulo. También puede [obtener la dirección IP externa en el panel de Kubernetes](azure-stack-edge-gpu-monitor-kubernetes-dashboard.md#get-ip-address-for-services-or-modules).


#### <a name="to-check-if-module-deployed-successfully"></a>Comprobación de si el módulo se ha implementado correctamente

Los módulos de proceso son contenedores que tienen una lógica de negocios implementada. Un pod de Kubernetes puede tener varios contenedores en ejecución. 

Para comprobar si un módulo de proceso se implementa correctamente, conéctese a la interfaz de PowerShell del dispositivo.
Ejecute el comando `get pods` y compruebe si se está ejecutando el contenedor (que corresponde al módulo de proceso).

Para obtener la lista de todos los pods que se ejecutan en un espacio de nombres específico, ejecute el siguiente comando:

`get pods -n <namespace>`

Para comprobar los módulos implementados con IoT Edge, ejecute el siguiente comando:

`get pods -n iotedge`

A continuación se muestra una salida de ejemplo de todos los pods que se ejecutan en el espacio de nombres `iotedge`.

```
[10.100.10.10]: PS>kubectl get pods -n iotedge
NAME                        READY   STATUS    RESTARTS   AGE
edgeagent-cf6d4ffd4-q5l2k   2/2     Running   0          20h
edgehub-8c9dc8788-2mvwv     2/2     Running   0          56m
filemove-66c49984b7-h8lxc   2/2     Running   0          56m
iotedged-675d7f4b5f-9nml4   1/1     Running   0          20h

[10.100.10.10]: PS>
```

El estado **Estado** indica que se están ejecutando todos los pods en el espacio de nombres y el estado **Listo** indica el número de contenedores implementados en un pod. En el ejemplo anterior, todos los pods están en ejecución y todos los módulos implementados en cada uno de los pods se están ejecutando. 

Para comprobar los módulos implementados mediante Azure Arc, ejecute el siguiente comando:

`get pods -n azure-arc`

Como alternativa, puede [conectarse al panel de Kubernetes para ver IoT Edge o implementaciones de Azure Arc](azure-stack-edge-gpu-monitor-kubernetes-dashboard.md#view-module-status).


Para obtener una salida más detallada de un pod específico para un espacio de nombres determinado, puede ejecutar el siguiente comando:

`kubectl describe pod <pod name> -n <namespace>` 

A continuación se muestra la salida de ejemplo.

```
[10.100.10.10]: PS>kubectl describe pod filemove-66c49984b7 -n iotedge
Name:           filemove-66c49984b7-h8lxc
Namespace:      iotedge
Priority:       0
Node:           k8s-1hwf613cl-1hwf613/10.139.218.12
Start Time:     Thu, 14 May 2020 12:46:28 -0700
Labels:         net.azure-devices.edge.deviceid=myasegpu-edge
                net.azure-devices.edge.hub=myasegpu2iothub.azure-devices.net
                net.azure-devices.edge.module=filemove
                pod-template-hash=66c49984b7
Annotations:    net.azure-devices.edge.original-moduleid: filemove
Status:         Running
IP:             172.17.75.81
IPs:            <none>
Controlled By:  ReplicaSet/filemove-66c49984b7
Containers:
    proxy:
    Container ID:   docker://fd7975ca78209a633a1f314631042a0892a833b7e942db2e7708b41f03e8daaf
    Image:          azureiotedge/azureiotedge-proxy:0.1.0-beta8
    Image ID:       docker://sha256:5efbf6238f13d24bab9a2b499e5e05bc0c33ab1587d6cf6f289cdbe7aa667563
    Port:           <none>
    Host Port:      <none>
    State:          Running
        Started:      Thu, 14 May 2020 12:46:30 -0700
    Ready:          True
    Restart Count:  0
    Environment:
        PROXY_LOG:  Debug
=============CUT===============================CUT===========================
Volumes:
    config-volume:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      iotedged-proxy-config
    Optional:  false
    trust-bundle-volume:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      iotedged-proxy-trust-bundle
    Optional:  false
    myasesmb1local:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  myasesmb1local
    ReadOnly:   false
    myasesmb1:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  myasesmb1
    ReadOnly:   false
    filemove-token-pzvw8:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  filemove-token-pzvw8
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                    node.kubernetes.io/unreachable:NoExecute for 300s
Events:          <none>


[10.100.10.10]: PS>
```

#### <a name="to-get-container-logs"></a>Para obtener registros de contenedores

Para obtener los registros de un módulo, ejecute el siguiente comando desde la interfaz de PowerShell del dispositivo:

`kubectl logs <pod_name> -n <namespace> --all-containers` 

Dado que la marca `all-containers` vuelca todos los registros de todos los contenedores, una buena manera de ver los errores recientes es usar la opción `--tail 10`.

A continuación se muestra una salida de ejemplo. 

```
[10.100.10.10]: PS>kubectl logs filemove-66c49984b7-h8lxc -n iotedge --all-containers --tail 10
DEBUG 2020-05-14T20:40:42Z: loop process - 0 events, 0.000s
DEBUG 2020-05-14T20:40:44Z: loop process - 0 events, 0.000s
DEBUG 2020-05-14T20:40:44Z: loop process - 0 events, 0.000s
DEBUG 2020-05-14T20:40:44Z: loop process - 1 events, 0.000s
DEBUG 2020-05-14T20:40:44Z: loop process - 0 events, 0.000s
DEBUG 2020-05-14T20:42:12Z: loop process - 0 events, 0.000s
DEBUG 2020-05-14T20:42:14Z: loop process - 0 events, 0.000s
DEBUG 2020-05-14T20:42:14Z: loop process - 0 events, 0.000s
DEBUG 2020-05-14T20:42:14Z: loop process - 1 events, 0.000s
DEBUG 2020-05-14T20:42:14Z: loop process - 0 events, 0.000s
05/14/2020 19:46:44: Info: Opening module client connection.
05/14/2020 19:46:45: Info: Open done.
05/14/2020 19:46:45: Info: Initializing with input: /home/input, output: /home/output, protocol: Amqp.
05/14/2020 19:46:45: Info: IoT Hub module client initialized.

[10.100.10.10]: PS>
```

### <a name="change-memory-processor-limits-for-kubernetes-worker-node"></a>Cambio de límites de memoria o procesador para el nodo de trabajo de Kubernetes

Para cambiar los límites de memoria o procesador para el nodo de trabajo de Kubernetes, siga estos pasos:

1. [Conexión a la interfaz de PowerShell del dispositivo](#connect-to-the-powershell-interface).
1. Para obtener los recursos actuales para el nodo de trabajo y las opciones de rol, ejecute el siguiente comando:

    `Get-AzureDataBoxEdgeRole`

    Esta es una salida de ejemplo. Tenga en cuenta los valores de `Name` y `Compute` en la sección `Resources`. `MemoryInBytes` y `ProcessorCount` indican los valores asignados actualmente de memoria y número de procesadores para el nodo de trabajo de Kubernetes.  

    ```powershell
    [10.100.10.10]: PS>Get-AzureDataBoxEdgeRole
    ImageDetail                : Name:mcr.microsoft.com/azureiotedge-agent
                                 Tag:1.0
                                 PlatformType:Linux
    EdgeDeviceConnectionString :
    IotDeviceConnectionString  :
    HubHostName                : ase-srp-007.azure-devices.net
    IotDeviceId                : srp-007-storagegateway
    EdgeDeviceId               : srp-007-edge
    Version                    :
    Id                         : 6ebeff9f-84c5-49a7-890c-f5e05520a506
    Name                       : IotRole
    Type                       : IOT
    Resources                  : Compute:
                                 MemoryInBytes:34359738368
                                 ProcessorCount:12
                                 VMProfile:
    
                                 Storage:
                                 EndpointMap:
                                 EndpointId:c0721210-23c2-4d16-bca6-c80e171a0781
                                 TargetPath:mysmbedgecloudshare1
                                 Name:mysmbedgecloudshare1
                                 Protocol:SMB
    
                                 EndpointId:6557c3b6-d3c5-4f94-aaa0-6b7313ab5c74
                                 TargetPath:mysmbedgelocalshare
                                 Name:mysmbedgelocalshare
                                 Protocol:SMB
                                 RootFileSystemStorageSizeInBytes:0
    
    HostPlatform               : KubernetesCluster
    State                      : Created
    PlatformType               : Linux
    HostPlatformInstanceId     : 994632cb-853e-41c5-a9cd-05b36ddbb190
    IsHostPlatformOwner        : True
    IsCreated                  : True    
    [10.100.10.10]: PS>
    ```
    
1. Para cambiar los valores de memoria y procesadores del nodo de trabajo, ejecute el siguiente comando:

    Set-AzureDataBoxEdgeRoleCompute -Name <Name value from the output of Get-AzureDataBoxEdgeRole> -Memory <Value in Bytes> -ProcessorCount <n.º de núcleos>

    Esta es una salida de ejemplo. 
    
    ```powershell
    [10.100.10.10]: PS>Set-AzureDataBoxEdgeRoleCompute -Name IotRole -MemoryInBytes 32GB -ProcessorCount 16
    
    ImageDetail                : Name:mcr.microsoft.com/azureiotedge-agent
                                 Tag:1.0
                                 PlatformType:Linux
    
    EdgeDeviceConnectionString :
    IotDeviceConnectionString  :
    HubHostName                : ase-srp-007.azure-devices.net
    IotDeviceId                : srp-007-storagegateway
    EdgeDeviceId               : srp-007-edge
    Version                    :
    Id                         : 6ebeff9f-84c5-49a7-890c-f5e05520a506
    Name                       : IotRole
    Type                       : IOT
    Resources                  : Compute:
                                 MemoryInBytes:34359738368
                                 ProcessorCount:16
                                 VMProfile:
    
                                 Storage:
                                 EndpointMap:
                                 EndpointId:c0721210-23c2-4d16-bca6-c80e171a0781
                                 TargetPath:mysmbedgecloudshare1
                                 Name:mysmbedgecloudshare1
                                 Protocol:SMB
    
                                 EndpointId:6557c3b6-d3c5-4f94-aaa0-6b7313ab5c74
                                 TargetPath:mysmbedgelocalshare
                                 Name:mysmbedgelocalshare
                                 Protocol:SMB
    
                                 RootFileSystemStorageSizeInBytes:0
    
    HostPlatform               : KubernetesCluster
    State                      : Created
    PlatformType               : Linux
    HostPlatformInstanceId     : 994632cb-853e-41c5-a9cd-05b36ddbb190
    IsHostPlatformOwner        : True
    IsCreated                  : True
    
    [10.100.10.10]: PS>    
    ```

Al cambiar el uso de la memoria y el procesador, siga estas instrucciones.

- La memoria predeterminada es el 25 % de la especificación del dispositivo.
- El número de procesadores predeterminado es el 30 % de la especificación del dispositivo.
- Al cambiar los valores de memoria y la cantidad de procesadores, se recomienda que varíe los valores entre el 15 % y el 60 % de la memoria del dispositivo y el número de procesadores. 
- Se recomienda un límite superior del 60 % para que haya suficientes recursos para los componentes del sistema. 

## <a name="connect-to-bmc"></a>Conexión al BMC

El controlador de administración de placa base (BMC) se usa para supervisar y administrar el dispositivo de forma remota. En esta sección se describen los cmdlets que se pueden usar para administrar la configuración de BMC. Antes de ejecutar cualquiera de estos cmdlets, [conéctese a la interfaz de PowerShell del dispositivo](#connect-to-the-powershell-interface).

- `Get-HcsNetBmcInterface`: use este cmdlet para obtener las propiedades de configuración de red del BMC, por ejemplo, `IPv4Address`, `IPv4Gateway`, `IPv4SubnetMask` `DhcpEnabled`. 
    
    Este es una salida de ejemplo:
    
    ```powershell
    [10.100.10.10]: PS>Get-HcsNetBmcInterface
    IPv4Address   IPv4Gateway IPv4SubnetMask DhcpEnabled
    -----------   ----------- -------------- -----------
    10.128.53.186 10.128.52.1 255.255.252.0        False
    [10.100.10.10]: PS>
    ```
- `Set-HcsNetBmcInterface`: puede usar este cmdlet de dos maneras.

    - Use el cmdlet para habilitar o deshabilitar la configuración de DHCP para BMC con el valor adecuado del parámetro `UseDhcp`. 

        ```powershell
        Set-HcsNetBmcInterface -UseDhcp $true
        ```

        Este es una salida de ejemplo: 

        ```powershell
        [10.100.10.10]: PS>Set-HcsNetBmcInterface -UseDhcp $true
        [10.100.10.10]: PS>Get-HcsNetBmcInterface
        IPv4Address IPv4Gateway IPv4SubnetMask DhcpEnabled
        ----------- ----------- -------------- -----------
        10.128.54.8 10.128.52.1 255.255.252.0         True
        [10.100.10.10]: PS>
        ```

    - Use este cmdlet para realizar la configuración estática del BMC. Puede especificar los valores de `IPv4Address`, `IPv4Gateway` y `IPv4SubnetMask`. 
    
        ```powershell
        Set-HcsNetBmcInterface -IPv4Address "<IPv4 address of the device>&quot; -IPv4Gateway &quot;<IPv4 address of the gateway>&quot; -IPv4SubnetMask &quot;<IPv4 address for the subnet mask>"
        ```        
        
        Este es una salida de ejemplo: 

        ```powershell
        [10.100.10.10]: PS>Set-HcsNetBmcInterface -IPv4Address 10.128.53.186 -IPv4Gateway 10.128.52.1 -IPv4SubnetMask 255.255.252.0
        [10.100.10.10]: PS>Get-HcsNetBmcInterface
        IPv4Address   IPv4Gateway IPv4SubnetMask DhcpEnabled
        -----------   ----------- -------------- -----------
        10.128.53.186 10.128.52.1 255.255.252.0        False
        [10.100.10.10]: PS>
        ```    

- `Set-HcsBmcPassword`: use este cmdlet para modificar la contraseña de BMC para `EdgeUser`. El nombre de usuario, `EdgeUser`, distingue mayúsculas de minúsculas.

    Este es una salida de ejemplo: 

    ```powershell
    [10.100.10.10]: PS> Set-HcsBmcPassword -NewPassword "Password1"
    [10.100.10.10]: PS>
    ```

## <a name="exit-the-remote-session"></a>Salir de la sesión remota

Para salir de la sesión remota de PowerShell, cierre la ventana de PowerShell.

## <a name="next-steps"></a>Pasos siguientes

- Implemente [Azure Stack Edge Pro GPU](azure-stack-edge-gpu-deploy-prep.md) en Azure Portal.