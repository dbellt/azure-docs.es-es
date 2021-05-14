---
title: Implementación de un controlador de datos de Azure Arc en un dispositivo Azure Stack Edge Pro con GPU | Microsoft Docs
description: Describe cómo implementar un controlador de datos de Azure Arc y Azure Data Services en el dispositivo Azure Stack Edge Pro con GPU.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 04/15/2021
ms.author: alkohli
ms.openlocfilehash: d56e03cd650032a775c30b02d939cf934f384fae
ms.sourcegitcommit: 590f14d35e831a2dbb803fc12ebbd3ed2046abff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107568614"
---
# <a name="deploy-azure-data-services-on-your-azure-stack-edge-pro-gpu-device"></a>Implementación de Azure Data Services en el dispositivo Azure Stack Edge Pro con GPU

[!INCLUDE [applies-to-GPU-and-pro-r-and-mini-r-skus](../../includes/azure-stack-edge-applies-to-gpu-pro-r-mini-r-sku.md)]

En este artículo se describe el proceso de creación de un controlador de datos de Azure Arc y, después, la implementación de Azure Data Services en el dispositivo Azure Stack Edge Pro con GPU. 

El controlador de datos de Azure Arc es el plano de control local que habilita Azure Data Services en los entornos administrados por el cliente. Una vez que haya creado el controlador de datos de Azure Arc en el clúster de Kubernetes que se ejecuta en el dispositivo Azure Stack Edge Pro GPU, puede implementar Azure Data Services como SQL Managed Instance (versión preliminar) en ese controlador de datos.

El procedimiento para crear un controlador de datos y, a continuación, implementar un SQL Managed Instance implica el uso de PowerShell y `kubectl`: una herramienta nativa que proporciona acceso de línea de comandos al clúster de Kubernetes en el dispositivo.


## <a name="prerequisites"></a>Requisitos previos

Antes de comenzar, asegúrese de que:

1. Tiene acceso a un dispositivo Azure Stack Edge Pro GPU y ha activado el dispositivo como se describe en [Activación de Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-activate.md).

1. Ha habilitado el rol de proceso en el dispositivo. También se ha creado un clúster de Kubernetes en el dispositivo al configurar el proceso en el dispositivo según las instrucciones de [Configuración del proceso en un dispositivo Azure Stack Edge Pro GPU](azure-stack-edge-gpu-deploy-configure-compute.md).

1. Tiene el punto de conexión de la API de Kubernetes de la página **Dispositivo** de la interfaz de usuario web local. Para obtener más información, consulte las instrucciones en [Obtención de los puntos de conexión de Kubernetes](azure-stack-edge-gpu-deploy-configure-compute.md#get-kubernetes-endpoints).

1. Tiene acceso a un cliente que se conectará a su dispositivo. 
    1. En este artículo se usa un sistema cliente de Windows que ejecuta PowerShell 5.0 o posterior para acceder al dispositivo. Puede tener otro cliente con un [sistema operativo compatible](azure-stack-edge-gpu-system-requirements.md#supported-os-for-clients-connected-to-device). 
    1. Instale `kubectl` en su cliente. Para la versión de cliente:
        1. Identifique la versión del servidor de Kubernetes instalada en el dispositivo. En la interfaz de usuario local del dispositivo, vaya a la página **Actualizaciones de software**. Tome nota de la **versión del servidor de Kubernetes** en esta página.
        1. Descargue un cliente que no esté distanciado más de una versión secundaria de la del maestro. Sin embargo, la versión del cliente puede superar a la del maestro en hasta una versión secundaria. Por ejemplo, un maestro v1.3 debe funcionar con nodos v1.1, v1.2 y v1.3, y debe funcionar con clientes v1.2, v1.3 y v1.4. Para obtener información sobre la versión del cliente de Kubernetes, consulte [Kubernetes version and version skew support policy](https://kubernetes.io/docs/setup/release/version-skew-policy/#supported-version-skew) (Versión de Kubernetes y directiva de soporte de asimetría de versiones).
    
1. Como alternativa, [instale las herramientas de cliente para implementar y administrar los servicios de datos habilitados para Azure Arc](../azure-arc/data/install-client-tools.md). Estas herramientas no son necesarias, pero están recomendadas.  
1. Asegúrese de que tiene suficientes recursos disponibles en el dispositivo para aprovisionar un controlador de datos y una instancia de SQL Managed Instance. Para el controlador de datos y la instancia de SQL Managed Instance, necesitará un mínimo de 16 GB de RAM y 4 núcleos de CPU. Para obtener instrucciones detalladas, visite [Requisitos mínimos para la implementación de los servicios de datos habilitados para Azure Arc](../azure-arc/data/sizing-guidance.md#minimum-deployment-requirements).


## <a name="configure-kubernetes-external-service-ips"></a>Configuración de IP del servicio externo de Kubernetes

1. Diríjase a la interfaz de usuario web local del dispositivo y, después, a **Proceso**.
1. Seleccione la red habilitada para proceso. 

    ![Página Proceso en la interfaz de usuario local 2](./media/azure-stack-edge-gpu-deploy-arc-data-controller/compute-network-1.png)

1. Asegúrese de proporcionar tres direcciones IP de servicio externo de Kubernetes adicionales (además de las direcciones IP que ya ha configurado para otros servicios o contenedores externos). El controlador de datos necesitará dos direcciones IP de servicio y la tercera dirección IP se usará al crear una instancia de SQL Managed Instance. Necesitará una dirección IP por cada servicio de datos adicional que implementará. 

    ![Página Proceso en la interfaz de usuario local 3](./media/azure-stack-edge-gpu-deploy-arc-data-controller/compute-network-2.png)

1. Aplique la configuración y estas nuevas direcciones IP surtirán efecto inmediatamente en un clúster de Kubernetes existente. 


## <a name="deploy-azure-arc-data-controller"></a>Implementación del controlador de datos de Azure Arc

Antes de implementar un controlador de datos, deberá crear un espacio de nombres.

### <a name="create-namespace"></a>Crear un espacio de nombres 

Cree un nuevo espacio de nombres dedicado en el que se implementará el controlador de datos. También debe crear un usuario y, a continuación, debe concederle acceso al espacio de nombres que creó. 

> [!NOTE]
> En el caso del espacio de nombres y los nombres de usuario, se aplican las [convenciones de nomenclatura del subdominio DNS](https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#dns-subdomain-names).

1. [Conéctese a la interfaz de PowerShell](azure-stack-edge-gpu-connect-powershell-interface.md#connect-to-the-powershell-interface).
1. Cree un espacio de nombres. Escriba:

    `New-HcsKubernetesNamespace -Namespace <Name of namespace>`

1. Cree un usuario. Escriba: 

    `New-HcsKubernetesUser -UserName <User name>`

1. Un archivo de configuración se muestra en texto sin formato. Copie este archivo y guárdelo como archivo *config*. 

    > [!IMPORTANT]
    > No guarde el archivo de configuración como archivo *.txt*, guarde el archivo sin ninguna extensión de archivo.

1. El archivo de configuración debe residir en la carpeta `.kube` del perfil de usuario en la máquina local. Copie el archivo en esa carpeta del perfil de usuario.

    ![Ubicación del archivo de configuración en el cliente](media/azure-stack-edge-gpu-create-kubernetes-cluster/location-config-file.png)
1. Conceda acceso al usuario al espacio de nombres que creó. Escriba: 

    `Grant-HcsKubernetesNamespaceAccess -Namespace <Name of namespace> -UserName <User name>`

    Este es un ejemplo de resultado de los comandos anteriores. En este ejemplo, se crea un espacio de nombres `myadstest`, un usuario `myadsuser` y se le concede al usuario acceso al espacio de nombres.
    
    ```powershell
    [10.100.10.10]: PS>New-HcsKubernetesNamespace -Namespace myadstest
    [10.100.10.10]: PS>New-HcsKubernetesUser -UserName myadsuser
    apiVersion: v1
    clusters:
    - cluster:
        certificate-authority-data: LS0tLS1CRUdJTiBD=======//snipped//=======VSVElGSUNBVEUtLS0tLQo=
        server: https://compute.myasegpudev.wdshcsso.com:6443
      name: kubernetes
    contexts:
    - context:
        cluster: kubernetes
        user: myadsuser
      name: myadsuser@kubernetes
    current-context: myadsuser@kubernetes
    kind: Config
    preferences: {}
    users:
    - name: myadsuser
      user:
        client-certificate-data: LS0tLS1CRUdJTiBDRV=========//snipped//=====EE9PQotLS0kFURSBLRVktLS0tLQo=
    
    [10.100.10.10]: PS>Grant-HcsKubernetesNamespaceAccess -Namespace myadstest -UserName myadsuser
    [10.100.10.10]: PS>Set-HcsKubernetesAzureArcDataController -SubscriptionId db4e2fdb-6d80-4e6e-b7cd-736098270664 -ResourceGroupName myasegpurg -Location "EastUS" -UserName myadsuser -Password "Password1" -DataControllerName "arctestcontroller" -Namespace myadstest
    [10.100.10.10]: PS>
    ```
1. Agregue una entrada DNS al archivo de hosts en el sistema. 

    1. Ejecute el Bloc de notas como administrador y abra el archivo `hosts` ubicado en `C:\windows\system32\drivers\etc\hosts`. 
    2. Use la información que guardó de la página **Dispositivo** en la interfaz de usuario local (requisito previo) para crear la entrada en el archivo de hosts. 

        Por ejemplo, copie este punto de conexión `https://compute.myasegpudev.microsoftdatabox.com/[10.100.10.10]` para crear la siguiente entrada con la dirección IP y el dominio DNS del dispositivo: 

        `10.100.10.10 compute.myasegpudev.microsoftdatabox.com`

1. Para comprobar que puede conectarse a los pods de Kubernetes, inicie un símbolo del sistema o una sesión de PowerShell. Escriba:
    
    ```powershell
    PS C:\WINDOWS\system32> kubectl get pods -n "myadstest"
    No resources found.
    PS C:\WINDOWS\system32>
    ```
Ahora puede implementar el controlador de datos y las aplicaciones de servicio de datos en el espacio de nombres y, a continuación, ver esas aplicaciones y sus registros.

### <a name="create-data-controller"></a>Creación de un controlador de datos

El controlador de datos es una colección de pods que se implementan en el clúster de Kubernetes para proporcionar una API, el servicio de controlador, el programa previo y las bases de datos y paneles de supervisión. Siga estos pasos para crear un controlador de datos en el clúster de Kubernetes que existe en el dispositivo Azure Stack Edge del espacio de nombres que creó anteriormente.   

1. Recopile la siguiente información que necesitará para crear un controlador de datos:

    
    |Columna1  |Columna2  |
    |---------|---------|
    |Nombre del controlador de datos     |Nombre descriptivo para el controlador de datos. Por ejemplo, `arctestdatacontroller`.         |
    |Nombre de usuario del controlador de datos     |Cualquier nombre de usuario del usuario administrador del controlador de datos. El nombre de usuario y la contraseña del controlador de datos se usan para autenticarse en la API del controlador de datos con el fin de realizar funciones administrativas.          |
    |Contraseña del controlador de datos     |Contraseña para el usuario administrador del controlador de datos. Elija una contraseña segura y compártala solo con aquellos usuarios que necesiten tener privilegios de administrador de clústeres.         |
    |Nombre del espacio de nombres de Kubernetes     |Nombre del espacio de nombres de Kubernetes en el que quiere crear el controlador de datos.         |
    |Identificador de suscripción de Azure     |GUID de la suscripción a Azure en el que quiere que se cree el recurso del controlador de datos en Azure.         |
    |Nombre del grupo de recursos de Azure     |Nombre del grupo de recursos en el que quiere que se cree el recurso del controlador de datos en Azure.         |
    |Ubicación de Azure     |Ubicación de Azure en la que se almacenarán los metadatos del recurso del controlador de datos en Azure. Para obtener una lista de las regiones disponibles, consulte Infraestructura global de Azure/Productos por región.|


1. Conéctese a la interfaz de PowerShell. Para crear el controlador de datos, escriba lo siguiente: 

    ```powershell
    Set-HcsKubernetesAzureArcDataController -SubscriptionId <Subscription ID> -ResourceGroupName <Resource group name> -Location <Location without spaces> -UserName <User you created> -Password <Password to authenticate to Data Controller> -DataControllerName <Data Controller Name> -Namespace <Namespace you created>    
    ```
    Este es un ejemplo de resultado de los comandos anteriores.

    ```powershell
    [10.100.10.10]: PS>Set-HcsKubernetesAzureArcDataController -SubscriptionId db4e2fdb-6d80-4e6e-b7cd-736098270664 -ResourceGroupName myasegpurg -Location "EastUS" -UserName myadsuser -Password "Password1" -DataControllerName "arctestcontroller" -Namespace myadstest
    [10.100.10.10]: PS> 
    ```
    
    La implementación tarda aproximadamente 5 minutos en completarse.

    > [!NOTE]
    > The data controller created on Kubernetes cluster on your Azure Stack Edge Pro GPU device works only in the disconnected mode in the current release. El modo desconectado es para el controlador de datos y no para el dispositivo.

### <a name="monitor-data-creation-status"></a>Supervisión del estado de creación de datos

1. Abra otra ventana de PowerShell.
1. Use el siguiente comando `kubectl` para supervisar el estado de creación del controlador de datos. 

    ```powershell
    kubectl get datacontroller/<Data controller name> --namespace <Name of your namespace>
    ```
    Cuando se crea el controlador, el estado debe ser `Ready`.
    Este es un ejemplo de resultado del comando anterior:

    ```powershell
    PS C:\WINDOWS\system32> kubectl get datacontroller/arctestcontroller --namespace myadstest
    NAME                STATE
    arctestcontroller   Ready
    PS C:\WINDOWS\system32>
    ```
1. Para identificar las direcciones IP asignadas a los servicios externos que se ejecutan en el controlador de datos, use el comando `kubectl get svc -n <namespace>`. Este es una salida de ejemplo:

    ```powershell
    PS C:\WINDOWS\system32> kubectl get svc -n myadstest
    NAME                      TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                                       AGE
    controldb-svc             ClusterIP      172.28.157.130   <none>        1433/TCP,8311/TCP,8411/TCP                    3d21h
    controller-svc            ClusterIP      172.28.123.251   <none>        443/TCP,8311/TCP,8301/TCP,8411/TCP,8401/TCP   3d21h
    controller-svc-external   LoadBalancer   172.28.154.30    10.57.48.63   30080:31090/TCP                               3d21h
    logsdb-svc                ClusterIP      172.28.52.196    <none>        9200/TCP,8300/TCP,8400/TCP                    3d20h
    logsui-svc                ClusterIP      172.28.85.97     <none>        5601/TCP,8300/TCP,8400/TCP                    3d20h
    metricsdb-svc             ClusterIP      172.28.255.103   <none>        8086/TCP,8300/TCP,8400/TCP                    3d20h
    metricsdc-svc             ClusterIP      172.28.208.191   <none>        8300/TCP,8400/TCP                             3d20h
    metricsui-svc             ClusterIP      172.28.158.163   <none>        3000/TCP,8300/TCP,8400/TCP                    3d20h
    mgmtproxy-svc             ClusterIP      172.28.228.229   <none>        443/TCP,8300/TCP,8311/TCP,8400/TCP,8411/TCP   3d20h
    mgmtproxy-svc-external    LoadBalancer   172.28.166.214   10.57.48.64   30777:30621/TCP                               3d20h
    sqlex-svc                 ClusterIP      None             <none>        1433/TCP                                      3d20h
    PS C:\WINDOWS\system32>
    ```

## <a name="deploy-sql-managed-instance"></a>Implementación de SQL Managed Instance

Después de haber creado correctamente el controlador de datos, puede usar una plantilla para implementar una instancia de SQL Managed Instance en el controlador de datos.

### <a name="deployment-template"></a>Plantilla de implementación

Use la siguiente plantilla de implementación para implementar una instancia de SQL Managed Instance en el controlador de datos del dispositivo.

```yml
apiVersion: v1
data:
    password: UGFzc3dvcmQx
    username: bXlhZHN1c2Vy
kind: Secret
metadata:
    name: sqlex-login-secret
type: Opaque
---
apiVersion: sql.arcdata.microsoft.com/v1alpha1
kind: sqlmanagedinstance
metadata:
    name: sqlex
spec:
    limits:
    memory: 4Gi
    vcores: "4"
    requests:
    memory: 2Gi
    vcores: "1"
    service:
    type: LoadBalancer
    storage:
    backups:
        className: ase-node-local
        size: 5Gi
    data:
        className: ase-node-local
        size: 5Gi
    datalogs:
        className: ase-node-local
        size: 5Gi
    logs:
        className: ase-node-local
        size: 1Gi
```


#### <a name="metadata-name"></a>Nombre de los metadatos
    
El nombre de metadatos es el nombre de la instancia de SQL Managed Instance. El pod asociado en el comando `deployment.yaml` anterior tendrá asignado el nombre `sqlex-n` (`n` es el número de pods asociados con la aplicación). 
    
#### <a name="password-and-username-data"></a>Datos de contraseña y nombre de usuario

El nombre de usuario y la contraseña del controlador de datos se usan para autenticarse en la API del controlador de datos con el fin de realizar funciones administrativas. El secreto de Kubernetes para el nombre de usuario y la contraseña del controlador de datos en la plantilla de implementación son cadenas codificadas en base64. 

Puede usar una herramienta en línea para codificar en Base64 el nombre de usuario y la contraseña que desee, o bien puede usar las herramientas integradas de la CLI, en función de la plataforma. Cuando use una herramienta de codificación base64 en línea, proporcione las cadenas de nombre de usuario y contraseña (que especificó al crear el controlador de datos) en la herramienta y esta generará las cadenas codificadas en base64 correspondientes.
    
#### <a name="service-type"></a>Tipo de servicio

El tipo de servicio debe establecerse en `LoadBalancer`.
    
#### <a name="storage-class-name"></a>Nombre de la clase de almacenamiento

Puede identificar la clase de almacenamiento en el dispositivo Azure Stack Edge que la implementación usará para los datos, las copias de seguridad, los registros de datos y los registros. Use el comando `kubectl get storageclass` para obtener la clase de almacenamiento implementada en el dispositivo.

```powershell
PS C:\WINDOWS\system32> kubectl get storageclass
NAME             PROVISIONER      RECLAIMPOLICY  VOLUMEBINDINGMODE     ALLOWVOLUMEEXPANSION   AGE
ase-node-local   rancher.io/local-path   Delete  WaitForFirstConsumer  false                  5d23h
PS C:\WINDOWS\system32>
```
En la salida de ejemplo anterior, la clase de almacenamiento `ase-node-local` del dispositivo debe especificarse en la plantilla.
 
#### <a name="spec"></a>Especificación

Para crear una instancia de SQL Managed Instance en el dispositivo Azure Stack Edge, puede especificar los requisitos de memoria y CPU en la sección spec de `deployment.yaml`. Cada instancia de SQL Managed Instance debe solicitar un mínimo de 2 GB de memoria y 1 núcleo de CPU, como se muestra en el ejemplo siguiente. 

```yml
spec:
    limits:
    memory: 4Gi
    vcores: "4"
    requests:
    memory: 2Gi
    vcores: "1"
```  

Para obtener instrucciones detalladas sobre el ajuste de tamaño para el controlador de datos y 1 instancia de SQL Managed Instance, revise los [detalles de cambio de tamaño de SQL Managed Instance](../azure-arc/data/sizing-guidance.md#sql-managed-instance-sizing-details).

### <a name="run-deployment-template"></a>Ejecución de la plantilla de implementación

Ejecute el archivo `deployment.yaml` con el comando siguiente:

```powershell
kubectl create -n <Name of namespace that you created> -f <Path to the deployment yaml> 
```

Este es un ejemplo de resultado del comando siguiente:

```powershell
PS C:\WINDOWS\system32> kubectl get pods -n "myadstest"
No resources found.
PS C:\WINDOWS\system32> 
PS C:\WINDOWS\system32> kubectl create -n myadstest -f C:\azure-arc-data-services\sqlex.yaml  secret/sqlex-login-secret created
sqlmanagedinstance.sql.arcdata.microsoft.com/sqlex created
PS C:\WINDOWS\system32> kubectl get pods --namespace myadstest
NAME                 READY   STATUS    RESTARTS   AGE
bootstrapper-mv2cd   1/1     Running   0          83m
control-w9s9l        2/2     Running   0          78m
controldb-0          2/2     Running   0          78m
controlwd-4bmc5      1/1     Running   0          64m
logsdb-0             1/1     Running   0          64m
logsui-wpmw2         1/1     Running   0          64m
metricsdb-0          1/1     Running   0          64m
metricsdc-fb5r5      1/1     Running   0          64m
metricsui-56qzs      1/1     Running   0          64m
mgmtproxy-2ckl7      2/2     Running   0          64m
sqlex-0              3/3     Running   0          13m
PS C:\WINDOWS\system32>
```

El pod `sqlex-0` en el resultado de ejemplo indica el estado de SQL Managed Instance.

## <a name="remove-data-controller"></a>Eliminación de un controlador de datos

Para quitar el controlador de datos, elimine el espacio de nombres dedicado en el que lo implementó.


```powershell
kubectl delete ns <Name of your namespace>
```


## <a name="next-steps"></a>Pasos siguientes

- [Implementación de una aplicación sin estado en Azure Stack Edge Pro](./azure-stack-edge-gpu-deploy-stateless-application-kubernetes.md).
