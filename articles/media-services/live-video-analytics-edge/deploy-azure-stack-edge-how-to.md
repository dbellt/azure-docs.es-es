---
title: Implementación de Live Video Analytics en Azure Stack Edge
description: En este artículo se indican los pasos que ayudan a implementar Live Video Analytics en Azure Stack Edge.
ms.topic: how-to
ms.date: 09/09/2020
ms.openlocfilehash: 3f751a69bf4d5efca80047479917b5f40e14fe3f
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108144734"
---
# <a name="deploy-live-video-analytics-on-azure-stack-edge"></a>Implementación de Live Video Analytics en Azure Stack Edge

En este artículo se indican los pasos que ayudan a implementar Live Video Analytics en Azure Stack Edge. Una vez configurado y activado el dispositivo, está listo para la implementación de Live Video Analytics. 

En lo que respecta a Live Video Analytics, se implementa mediante IoT Hub, aunque los recursos de Azure Stack Edge exponen una API de Kubernetes que permite al cliente implementar soluciones adicionales no compatibles con IoT Hub que pueden interactuar con Live Video Analytics. 

> [!TIP]
> El uso de la API de Kubernetes (K8s) para la implementación personalizada es un caso avanzado. Se recomienda que el cliente cree módulos perimetrales e implemente mediante IoT Hub para cada recurso de Azure Stack Edge en lugar de usar la API de Kubernetes. En este artículo se muestran los pasos necesarios para implementar el módulo Live Video Analytics mediante IoT Hub.

## <a name="prerequisites"></a>Requisitos previos

* Suscripción de Azure en la que tenga [privilegios de propietario](../../role-based-access-control/built-in-roles.md#owner).
* Un recurso de [Azure Stack Edge](../../databox-online/azure-stack-edge-gpu-deploy-prep.md).
   
* [Una instancia de IoT Hub](../../iot-hub/iot-hub-create-through-portal.md).
* Una [entidad de servicio](./create-custom-azure-resource-manager-role-how-to.md#create-service-principal) para el módulo Live Video Analytics.

   Use alguna de estas regiones en las que IoT Hub está disponible: Este de EE. UU. 2, Centro de EE. UU., Centro-norte de EE. UU., Japón Oriental, Oeste de EE. UU. 2, Centro-oeste de EE. UU., Este de Canadá, Sur de Reino Unido, Centro de Francia, Sur de Francia, Norte de Suiza, Oeste de Suiza y Japón Occidental.
* Cuenta de almacenamiento

    Se recomienda usar cuentas de almacenamiento V2 de uso general (GPv2).  
    Obtenga más información sobre las [cuentas de almacenamiento v2 de uso general](../../storage/common/storage-account-upgrade.md?tabs=azure-portal).
* [Visual Studio Code](https://code.visualstudio.com/) en la máquina de desarrollo. Asegúrese de tener la [extensión Azure IoT Tools](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools).
* Asegúrese de que la red a la que está conectada la máquina de desarrollo permita Advanced Message Queueing Protocol a través del puerto 5671. Esta configuración permite a Azure IoT Tools comunicarse con Azure IoT Hub.

## <a name="configuring-azure-stack-edge-for-using-live-video-analytics"></a>Configuración de Azure Stack Edge para usar Live Video Analytics

Azure Stack Edge es una solución de hardware como servicio y un dispositivo informático perimetral habilitado para inteligencia artificial que cuenta con capacidades de transferencia de datos de red. Lea más sobre [Azure Stack Edge y vea instrucciones de configuración detalladas](../../databox-online/azure-stack-edge-gpu-deploy-prep.md). Para comenzar, siga las instrucciones de los vínculos siguientes:

* [Creación de recursos de Data Box Gateway o Azure Stack Edge](../../databox-online/azure-stack-edge-gpu-deploy-prep.md?tabs=azure-portal#create-a-new-resource)
* [Instalación y configuración](../../databox-online/azure-stack-edge-gpu-deploy-install.md)
* Conexión y activación

    1. [Conexión](../../databox-online/azure-stack-edge-gpu-deploy-connect.md)
    2. [Configuración de la red](../../databox-online/azure-stack-edge-gpu-deploy-configure-network-compute-web-proxy.md)
    3. [Configuración de dispositivos](../../databox-online/azure-stack-edge-gpu-deploy-set-up-device-update-time.md)
    4. [Configurar certificados](../../databox-online/azure-stack-edge-gpu-deploy-configure-certificates.md)
    5. [Activar](../../databox-online/azure-stack-edge-gpu-deploy-activate.md)
* [Asociación de una instancia de IoT Hub a Azure Stack Edge](../../databox-online/azure-stack-edge-gpu-deploy-configure-compute.md#configure-compute)
### <a name="enable-compute-prerequisites-on-the-azure-stack-edge-local-ui"></a>Habilitación de los requisitos previos de proceso en la interfaz de usuario local de Azure Stack Edge

Antes de continuar, asegúrese de que:

* Ha activado el recurso de Azure Stack Edge.
* Tiene acceso a un sistema cliente Windows con PowerShell 5.0 o posterior para acceder al recurso de Azure Stack Edge.
* Para implementar un clúster de Kubernetes, debe configurar el recurso de Azure Stack Edge mediante su [interfaz de usuario web local](../../databox-online/azure-stack-edge-deploy-connect-setup-activate.md#connect-to-the-local-web-ui-setup). 

    * Conecte y configure:
    
        1. [Conexión](../../databox-online/azure-stack-edge-gpu-deploy-connect.md)
        2. [Configuración de la red](../../databox-online/azure-stack-edge-gpu-deploy-configure-network-compute-web-proxy.md)
        3. [Configuración de dispositivos](../../databox-online/azure-stack-edge-gpu-deploy-set-up-device-update-time.md)
        4. [Configurar certificados](../../databox-online/azure-stack-edge-gpu-deploy-configure-certificates.md)
        5. [Activar](../../databox-online/azure-stack-edge-gpu-deploy-activate.md)
    * Para habilitar el proceso, vaya a la página Proceso de la interfaz de usuario web local del dispositivo.
    
        * Seleccione la interfaz de red que quiere habilitar para el proceso. Seleccione Habilitar. Al hacerlo, se crea un conmutador virtual para esa interfaz de red en el dispositivo.
        * Deje en blanco las direcciones IP de los nodos de prueba de Kubernetes y las de los servicios externos de Kubernetes.
        * Seleccione Aplicar: esta operación tarda unos 2 minutos.
        
        > [!div class="mx-imgBorder"]
        > :::image type="content" source="../../databox-online/media/azure-stack-edge-gpu-deploy-configure-network-compute-web-proxy/compute-network-2.png" alt-text="Requisitos previos de proceso en la interfaz de usuario local de Azure Stack Edge":::

        * Si el DNS no está configurado para la API de Kubernetes y el recurso de Azure Stack Edge, puede actualizar el archivo host de Windows.
        
            * Abra un editor de texto como Administrador
            * Abra el archivo "to C:\Windows\System32\drivers\etc\hosts"
            * Agregue la IPv4 y el nombre de host del nombre de dispositivo de la API de Kubernetes al archivo. (Esta información puede encontrarse en el portal de Azure Stack Edge, en la sección Dispositivos).
            * Guardar y cerrar

### <a name="deploy-live-video-analytics-edge-module-using-azure-portal"></a>Implementación del módulo de Edge Live Video Analytics mediante Azure Portal

Para ello, solo vamos a realizar pasos concretos de [Implementación de Live Video Analytics mediante IoT Hub](deploy-iot-edge-device.md).

1. Omita los pasos de creación de grupos y usuarios y vaya a [Implementación del módulo de Edge Live Video Analytics](deploy-iot-edge-device.md#deploy-live-video-analytics-edge-module). Siga los pasos que se indican a continuación.
1. En Opciones de creación del contenedor, no es necesario establecer variables de entorno. Por lo tanto, omita este paso.
1. Abra la pestaña Opciones de creación del contenedor.

   * Copie y pegue el siguiente código JSON en el cuadro para limitar el tamaño de los archivos de registro que genera el módulo.
    
      ```json
        "HostConfig": {
                    "LogConfig": {
                        "Type": "",
                        "Config": {
                        "max-size": "10m",
                        "max-file": "10"
                        }
                    },
                    "Binds": [
                        "/var/media/:/var/media/",
                        "/var/lib/azuremediaservices:/var/lib/azuremediaservices"
                    ]
                    }
      ```
    
      > [!NOTE]
      > Si usa el protocolo gRPC con transferencia de memoria compartida, use el modo de IPC de host para el acceso a la memoria compartida entre las soluciones de inferencia y Live Video Analytics.
   
      ```json
          "HostConfig": {
                        "LogConfig": {
                            "Type": "",
                            "Config": {
                            "max-size": "10m",
                            "max-file": "10"`
                            }
                        },
                        "Binds": [
                            "/var/media/:/var/media/",
                            "/var/lib/azuremediaservices:/var/lib/azuremediaservices"
                        ],
                        "IpcMode": "host",
                        "ShmSize": 1536870912
                    }
      ```

      > [!NOTE]
      > La sección "Binds" del archivo JSON tiene dos entradas. LVA creará automáticamente los directorios que se mencionan en la sección de enlace anterior.  
        Puede actualizar los enlaces de dispositivos perimetrales, pero si lo hace, asegúrese de que esos directorios existan en el dispositivo.
    
    * "/var/lib/azuremediaservices:/var/lib/azuremediaservices": se usa para enlazar los datos de configuración persistentes de la aplicación desde el contenedor y almacenarlos en el dispositivo perimetral.
    * "/var/media:/var/media": esto enlaza las carpetas de elementos multimedia entre el dispositivo perimetral y el contenedor. Se usa para almacenar las grabaciones de vídeo cuando se ejecuta una topología de gráficos multimedia que admite el almacenamiento de clips de vídeo en el dispositivo perimetral.
        
1. Continúe con los pasos del documento y rellene la configuración del módulo gemelo.
1. Seleccione **Siguiente**: Rutas para continuar con la sección de rutas. Especifique las rutas.

    Mantenga las rutas predeterminadas y seleccione Siguiente: Revisar y crear para continuar en la sección de revisión.
1. [Revise y verifique la implementación](deploy-iot-edge-device.md#review-deployment)

#### <a name="optional-setup-docker-volume-mounts"></a>(Opcional) Configuración de montajes de volumen de Docker

Si quiere ver los datos de los directorios de trabajo, siga estos pasos para configurar montajes de volumen de Docker antes de la implementación. 

En estos pasos se explica cómo crear un usuario de puerta de enlace y configurar recursos compartidos de archivos para ver el contenido del directorio de trabajo de Live Video Analytics y la carpeta multimedia de Live Video Analytics.

> [!NOTE]
> Se admiten montajes de enlace, pero los montajes de volumen permiten que se vean los datos y, si se quiere, que se copien de forma remota. Es posible usar montajes de enlace y de volumen, pero no pueden apuntar a la misma ruta de acceso de contenedor.

1. Abra Azure Portal y vaya al recurso de Azure Stack Edge.
1. Cree un **Usuario de puerta de enlace** que pueda acceder a recursos compartidos.
    
    1. En el panel de navegación izquierdo, haga clic en **Cloud storage gateway** (Puerta de enlace de almacenamiento en la nube).
    1. Haga clic en **Users** (Usuarios) en el panel de navegación izquierdo.
    1. Haga clic en el icono **+ Add User** (+ Agregar usuario) para establecer el nombre de usuario y la contraseña. (Recomendado: `lvauser`).
    1. Haga clic en **Agregar**.
    
1. Cree un **Recurso compartido local** para la persistencia de Live Video Analytics.

    1. Haga clic en **Cloud storage gateway->Shares** (Puerta de enlace de almacenamiento en la nube > Recursos compartidos).
    1. Haga clic en **+ Agregar recursos compartidos**.
    1. Establezca un nombre de recurso compartido. (Recomendado: `lva`).
    1. Mantenga el tipo de recurso compartido como SMB.
    1. Asegúrese de que la opción **Usar el recurso compartido con el proceso perimetral** está activada.
    1. Asegúrese de que la opción **Configurar como recurso compartido local de Edge** está activada.
    1. En Detalles de usuario, conceda acceso al recurso compartido al usuario creado recientemente.
    1. Haga clic en **Crear**.
        
    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/deploy-azure-stack-edge-how-to/local-share.png" alt-text="Recurso compartido local":::  

    > [!TIP]
    > Con el cliente Windows conectado a Azure Stack Edge, conéctese a los recursos compartidos de SMB mediante los pasos [mencionados en este documento](../../databox-online/azure-stack-edge-deploy-add-shares.md#connect-to-an-smb-share).    

1. Cree un recurso compartido remoto para el almacenamiento de File Sync.

    1. En primer lugar, cree una cuenta de almacenamiento de blobs en la misma región. Para ello, haga clic en **Cloud storage gateway->Storage accounts** (Puerta de enlace de almacenamiento en la nube -> Cuentas de almacenamiento).
    1. Haga clic en **Cloud storage gateway->Shares** (Puerta de enlace de almacenamiento en la nube > Recursos compartidos).
    1. Haga clic en **+ Agregar recursos compartidos**.
    1. Establezca un nombre de recurso compartido. (Recomendado: media).
    1. Mantenga el tipo de recurso compartido como SMB.
    1. Asegúrese de que la opción **Usar el recurso compartido con el proceso perimetral** está activada.
    1. Asegúrese de que la opción **Configurar como recurso compartido local de Edge** no está activada.
    1. Seleccione la cuenta de almacenamiento creada recientemente.
    1. Establezca un nombre de contenedor.
    1. Establezca el tipo de almacenamiento en blob en bloques.
    1. En Detalles de usuario, conceda acceso al recurso compartido al usuario creado recientemente.
    1. Haga clic en **Crear**.    
    
    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/deploy-azure-stack-edge-how-to/remote-share.png" alt-text="Recurso compartido remoto":::
    
    
1. Actualice las opciones de creación de contenedor del módulo de Edge Live Video Analytics (vea el punto 4 del [documento sobre incorporación de módulos](deploy-iot-edge-device.md#add-modules)) para usar montajes de volumen.

   ```json
      "createOptions": 
         {
             "HostConfig": 
             {
                 "Binds": 
                 [
                     "/var/lib/azuremediaservices:/var/lib/azuremediaservices"
                 ],
                 "Mounts": 
                 [
                     {
                         "Target": "/var/media",
                         "Source": "media",
                         "Type": "volume"
                     }
                 ]
             }
         }
    ```

### <a name="verify-that-the-module-is-running"></a>Comprobación de que el módulo se está ejecutando

El último paso es asegurarse de que el módulo está conectado y funciona según lo previsto. El estado del entorno de ejecución del módulo debe ser "en ejecución" para el dispositivo IoT Edge del recurso de IoT Hub.

Para comprobar que el módulo se está ejecutando, haga lo siguiente:

1. En Azure Portal, vuelva al recurso de Azure Stack Edge.
1. Seleccione el elemento Módulos. Esta acción le lleva a la hoja Módulos. En la lista de módulos, identifique el módulo que ha implementado. El estado del entorno de ejecución del módulo que se agrega es En ejecución.

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/deploy-azure-stack-edge-how-to/iot-edge-custom-module.png" alt-text="Módulo personalizado":::

### <a name="configure-the-azure-iot-tools-extension"></a>Configuración de la extensión Azure IoT Tools

Siga estas instrucciones para conectarse a su centro de IoT mediante la extensión Azure IoT Tools.

1. En Visual Studio Code, seleccione Ver > Explorer. O bien, seleccione Ctrl+Mayús+E.
1. En la esquina inferior izquierda de la pestaña Explorador, seleccione Azure IoT Hub.
1. Seleccione el icono Más opciones para ver el menú contextual. Luego, seleccione Set IoT Hub Connection String (Establecer cadena de conexión de IoT Hub).
1. Cuando aparezca un cuadro de entrada, escriba la cadena de conexión de IoT Hub. 

   * Para obtener la cadena de conexión, vaya a la instancia de IoT Hub en Azure Portal y haga clic en Directivas de acceso compartido en el panel de navegación izquierdo.
   * Haga clic en iothubowner para obtener las claves de acceso compartido.
   * Copie la cadena de conexión: clave principal y péguela en el cuadro de entrada de VSCode.

   La cadena de conexión tiene el siguiente aspecto:<br/>`HostName=xxx.azure-devices.net;SharedAccessKeyName=iothubowner;SharedAccessKey=xxx`
    
   Si la conexión se realiza correctamente, aparece la lista de dispositivos perimetrales. Debería ver la instancia de Azure Stack Edge. Ahora puede administrar los dispositivos IoT Edge e interactuar con Azure IoT Hub mediante el menú contextual. Para ver los módulos implementados en el dispositivo perimetral, en el dispositivo de Azure Stack, expanda el nodo Módulos.
    
## <a name="troubleshooting"></a>Solución de problemas

* **Acceso a la API de Kubernetes (kubectl)**

    * Siga la documentación para configurar la máquina para el [acceso al clúster de Kubernetes](../../databox-online/azure-stack-edge-gpu-create-kubernetes-cluster.md).
    * Todos los módulos de IoT Edge implementados usan el espacio de nombres `iotedge`. Asegúrese de incluirlo al usar kubectl.  

* **Registros del módulo**

    La herramienta `iotedge` no es accesible para obtener registros. Debe usar [registros de kubectl](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#logs) para ver los registros o canalizar a un archivo. Ejemplo: <br/>  `kubectl logs deployments/mediaedge -n iotedge --all-containers`  

* **Métricas de pod y nodo**

    Use [kubectl top](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#top) para ver métricas de pod y nodo.
    <br/>`kubectl top pods -n iotedge` 

* **Redes de módulo**   
Para la detección de un módulo en Azure Stack Edge es necesario que el módulo tenga el enlace de puerto de host en createOptions. Entonces el módulo se puede direccionar a través de `moduleName:hostport`.
    
    ```json
    "createOptions": {
        "HostConfig": {
            "PortBindings": {
                "8554/tcp": [ { "HostPort": "8554" } ]
            }
        }
    }
    ```
    
* **Montaje de volumen**

    Un módulo no se inicia si el contenedor intenta montar un volumen en un directorio existente y que no esté vacío.

* **Memoria compartida al usar gRPC**

    La memoria compartida en recursos de Azure Stack Edge se admite en pods de cualquier espacio de nombres mediante IPC de host.
    Configuración de memoria compartida en un módulo perimetral para la implementación mediante IoT Hub.
    
    ```
    ...
    "createOptions": {
        "HostConfig": {
            "IpcMode": "host"
        }
    ...
        
    //(Advanced) Configuring shared memory on a K8s Pod or Deployment manifest for deployment via K8s API
    spec:
        ...
        template:
        spec:
            hostIPC: true
        ...
    ```
    
* **Colocalización de pod (avanzado)**

    Al usar K8s para implementar soluciones de inferencia personalizadas que se comunican con Live Video Analytics a través de gRPC, debe asegurarse de que los pods estén implementados en los mismos nodos que los módulos de Live Video Analytics.

    * **Opción 1**: usar afinidad de nodo y etiquetas de nodo integradas para la colocalización.

    Actualmente, la configuración personalizada de NodeSelector no parece ser una opción, ya que los usuarios no tienen acceso para establecer etiquetas en los nodos. Pero, en función de la topología del cliente y las convenciones de nomenclatura, es posible que puedan usar [etiquetas de nodo integradas](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#built-in-node-labels). Se puede agregar una sección nodeAffinity que haga referencia a los recursos de Azure Stack Edge con Live Video Analytics al manifiesto del pod de inferencia para lograr la colocalización.
    * **Opción 2**: usar afinidad de pod para la colocalización (recomendado).
Kubernetes es compatible con la [afinidad de pod](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#inter-pod-affinity-and-anti-affinity), que puede programar pods en el mismo nodo. Se puede agregar una sección podAffinity que haga referencia al módulo de Live Video Analytics al manifiesto del pod de inferencia para lograr la colocalización.

    ```json   
    // Example Live Video Analytics module deployment match labels
    selector:
      matchLabels:
        net.azure-devices.edge.deviceid: dev-ase-1-edge
        net.azure-devices.edge.module: mediaedge
    
    // Example Inference deployment manifest pod affinity
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: net.azure-devices.edge.module
                operator: In
                values:
                - mediaedge
            topologyKey: "kubernetes.io/hostname"
    ```
* **Código de error 404 al usar el módulo `rtspsim`**  
El contenedor leerá vídeos de una carpeta exactamente dentro del contenedor. Si asigna o enlaza una carpeta externa con la que ya existe dentro de la imagen de contenedor, Docker ocultará los archivos presentes en la imagen de contenedor.  
 
    Por ejemplo, sin enlaces, el contenedor puede tener estos archivos:  
    ```
    root@rtspsim# ls /live/mediaServer/media  
    /live/mediaServer/media/camera-300s.mkv  
    /live/mediaServer/media/win10.mkv  
    ```
     
    Y el host puede tener estos archivos:
    ```    
    C:\MyTestVideos> dir
    Test1.mkv
    Test2.mkv
    ```
     
    Sin embargo, cuando se agrega el siguiente enlace en el archivo de manifiesto de implementación, Docker sobrescribirá el contenido de/live/mediaServer/media para que coincida con lo que hay en el host.
    `C:\MyTestVideos:/live/mediaServer/media`
    
    ```
    root@rtspsim# ls /live/mediaServer/media
    /live/mediaServer/media/Test1.mkv
    /live/mediaServer/media/Test2.mkv
    ```

## <a name="next-steps"></a>Pasos siguientes

Puede usar el módulo para analizar secuencias de vídeo en directo mediante la invocación de métodos directos. [Invoque los métodos directos](get-started-detect-motion-emit-events-quickstart.md#use-direct-method-calls) en el módulo.