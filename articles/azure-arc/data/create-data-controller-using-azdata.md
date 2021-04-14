---
title: Creación de un controlador de datos con la CLI de datos de Azure (azdata)
description: Cree un controlador de datos de Azure Arc en un típico clúster de Kubernetes de varios nodos que ya haya creado mediante la CLI de datos de Azure (azdata).
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
ms.date: 04/07/2021
ms.topic: how-to
ms.openlocfilehash: f7bc90f2748d230ad50868cff5d7a8f7b69d850a
ms.sourcegitcommit: d40ffda6ef9463bb75835754cabe84e3da24aab5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "107029546"
---
# <a name="create-azure-arc-data-controller-using-the-azure-data-cli-azdata"></a>Creación de un controlador de datos de Azure Arc mediante [!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)]

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="prerequisites"></a>Requisitos previos

Revise el tema [Creación del controlador de datos de Azure Arc](create-data-controller.md) para obtener información general.

Para crear el controlador de datos de Azure Arc mediante [!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)], deberá tener [!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)] instalado.

   [Instale [!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)]](install-client-tools.md).

Independientemente de la plataforma de destino que elija, tendrá que establecer las siguientes variables de entorno antes de la creación para el usuario administrador del controlador de datos. Puede proporcionar estas credenciales a otras personas que necesiten tener acceso de administrador al controlador de datos según sea necesario.

**AZDATA_USERNAME**: nombre de usuario de su elección para el usuario administrador del controlador de datos. Ejemplo: `arcadmin`

**AZDATA_PASSWORD**: contraseña de su elección para el usuario administrador del controlador de datos. La contraseña debe tener al menos 8 caracteres y contener caracteres de tres de los siguientes cuatro conjuntos: mayúsculas, minúsculas, números y símbolos.

### <a name="linux-or-macos"></a>Linux o macOS

```console
export AZDATA_USERNAME="<your username of choice>"
export AZDATA_PASSWORD="<your password of choice>"
```

### <a name="windows-powershell"></a>Windows PowerShell

```console
$ENV:AZDATA_USERNAME="<your username of choice>"
$ENV:AZDATA_PASSWORD="<your password of choice>"
```

Tendrá que conectarse a un clúster de Kubernetes y autenticarse en él, además de tener un contexto de Kubernetes existente seleccionado antes de comenzar la creación del controlador de datos de Azure Arc. La forma en que se conecta a un clúster o servicio de Kubernetes varía; consulte la documentación de la distribución o el servicio de Kubernetes que usa para saber cómo conectarse al servidor de API de Kubernetes.

Puede comprobar que tiene una conexión de Kubernetes y confirmar el contexto actual con los siguientes comandos.

```console
kubectl get namespace
kubectl config current-context
```

## <a name="create-the-azure-arc-data-controller"></a>Creación del controlador de datos de Azure Arc

> [!NOTE]
> Puede usar un valor diferente para el parámetro `--namespace` del comando azdata arc dc create en los ejemplos siguientes, pero asegúrese de usar ese nombre de espacio de nombres en el parámetro `--namespace parameter` de los demás comandos siguientes.

- [Creación en Azure Kubernetes Service (AKS)](#create-on-azure-kubernetes-service-aks)
- [Creación en el motor de AKS en Azure Stack Hub](#create-on-aks-engine-on-azure-stack-hub)
- [Creación en AKS en Azure Stack HCI](#create-on-aks-on-azure-stack-hci)
- [Creación en Red Hat OpenShift en Azure (ARO)](#create-on-azure-red-hat-openshift-aro)
- [Creación en Red Hat OpenShift Container Platform (OCP)](#create-on-red-hat-openshift-container-platform-ocp)
- [Creación en Kubernetes ascendente de código abierto (kubeadm)](#create-on-open-source-upstream-kubernetes-kubeadm)
- [Creación en Elastic Kubernetes Service (EKS) de AWS](#create-on-aws-elastic-kubernetes-service-eks)
- [Creación en Kubernetes Engine Service (GKE) de Google Cloud](#create-on-google-cloud-kubernetes-engine-service-gke)

### <a name="create-on-azure-kubernetes-service-aks"></a>Creación en Azure Kubernetes Service (AKS)

De forma predeterminada, el perfil de implementación de AKS usa la clase de almacenamiento `managed-premium`. La clase de almacenamiento `managed-premium` solo funcionará si tiene máquinas virtuales que se implementaron mediante imágenes de máquina virtual con discos premium.

Si va a usar `managed-premium` como clase de almacenamiento, puede ejecutar el siguiente comando para implementar el controlador de datos. Sustituya los marcadores de posición del comando por el nombre del grupo de recursos, el identificador de la suscripción y la ubicación de Azure.

```console
azdata arc dc create --profile-name azure-arc-aks-premium-storage --namespace arc --name arc --subscription <subscription id> --resource-group <resource group name> --location <location> --connectivity-mode indirect

#Example:
#azdata arc dc create --profile-name azure-arc-aks-premium-storage --namespace arc --name arc --subscription 1e5ff510-76cf-44cc-9820-82f2d9b51951 --resource-group my-resource-group --location eastus --connectivity-mode indirect
```

Si no está seguro de qué clase de almacenamiento debe utilizar, indique la clase de almacenamiento `default`, que es compatible con cualquier tipo de máquina virtual que esté usando. No le ofrecerá el rendimiento más rápido.

Si desea usar la clase de almacenamiento `default`, puede ejecutar este comando:

```console
azdata arc dc create --profile-name azure-arc-aks-default-storage --namespace arc --name arc --subscription <subscription id> --resource-group <resource group name> --location <location> --connectivity-mode indirect

#Example:
#azdata arc dc create --profile-name azure-arc-aks-default-storage --namespace arc --name arc --subscription 1e5ff510-76cf-44cc-9820-82f2d9b51951 --resource-group my-resource-group --location eastus --connectivity-mode indirect
```

Una vez ejecutado el comando, continúe en [Supervisión del estado de creación](#monitoring-the-creation-status).

### <a name="create-on-aks-engine-on-azure-stack-hub"></a>Creación en el motor de AKS en Azure Stack Hub

De forma predeterminada, el perfil de implementación utiliza la clase de almacenamiento `managed-premium`. La clase de almacenamiento `managed-premium` solo funcionará si tiene máquinas virtuales de trabajo que se implementaron mediante imágenes de máquina virtual con discos premium en Azure Stack Hub.

Puede ejecutar el siguiente comando para implementar el controlador de datos con la clase de almacenamiento managed-premium:

```console
azdata arc dc create --profile-name azure-arc-aks-premium-storage --namespace arc --name arc --subscription <subscription id> --resource-group <resource group name> --location <location> --connectivity-mode indirect

#Example:
#azdata arc dc create --profile-name azure-arc-aks-premium-storage --namespace arc --name arc --subscription 1e5ff510-76cf-44cc-9820-82f2d9b51951 --resource-group my-resource-group --location eastus --connectivity-mode indirect
```

Si no está seguro de qué clase de almacenamiento debe utilizar, indique la clase de almacenamiento `default`, que es compatible con cualquier tipo de máquina virtual que esté usando. En Azure Stack Hub, los discos premium y los discos estándar cuentan con el respaldo de la misma infraestructura de almacenamiento. Por lo tanto, se espera que proporcionen el mismo rendimiento general, pero con distintos límites de IOPS.

Si desea usar la clase de almacenamiento `default`, puede ejecutar este comando.

```console
azdata arc dc create --profile-name azure-arc-aks-default-storage --namespace arc --name arc --subscription <subscription id> --resource-group <resource group name> --location <location> --connectivity-mode indirect

#Example:
#azdata arc dc create --profile-name azure-arc-aks-default-storage --namespace arc --name arc --subscription 1e5ff510-76cf-44cc-9820-82f2d9b51951 --resource-group my-resource-group --location eastus --connectivity-mode indirect
```

Una vez ejecutado el comando, continúe en [Supervisión del estado de creación](#monitoring-the-creation-status).

### <a name="create-on-aks-on-azure-stack-hci"></a>Creación en AKS en Azure Stack HCI

De forma predeterminada, el perfil de implementación utiliza una clase de almacenamiento denominada `default` y el tipo de servicio `LoadBalancer`.

Puede ejecutar el siguiente comando para implementar el controlador de datos con la clase de almacenamiento `default` y el tipo de servicio `LoadBalancer`.

```console
azdata arc dc create --profile-name azure-arc-aks-hci --namespace arc --name arc --subscription <subscription id> --resource-group <resource group name> --location <location> --connectivity-mode indirect

#Example:
#azdata arc dc create --profile-name azure-arc-aks-hci --namespace arc --name arc --subscription 1e5ff510-76cf-44cc-9820-82f2d9b51951 --resource-group my-resource-group --location eastus --connectivity-mode indirect
```

Una vez ejecutado el comando, continúe en [Supervisión del estado de creación](#monitoring-the-creation-status).


### <a name="create-on-azure-red-hat-openshift-aro"></a>Creación en Red Hat OpenShift en Azure (ARO)

Red Hat OpenShift en Azure requiere una restricción del contexto de seguridad.

#### <a name="apply-the-security-context"></a>Aplicación del contexto de seguridad

Antes de crear el controlador de datos en Red Hat OpenShift en Azure, deberá aplicar restricciones de contexto de seguridad (SCC) específicas. En el caso de la versión preliminar, se relajan las restricciones de seguridad. Las versiones futuras proporcionarán SCC actualizadas.

[!INCLUDE [apply-security-context-constraint](includes/apply-security-context-constraint.md)]

#### <a name="create-custom-deployment-profile"></a>Creación de un perfil de implementación personalizado

Use el perfil `azure-arc-azure-openshift` para Azure RedHat Open Shift.

```console
azdata arc dc config init --source azure-arc-azure-openshift --path ./custom
```

#### <a name="create-data-controller"></a>Creación de un controlador de datos

Puede ejecutar el comando siguiente para crear el controlador de datos:

> [!NOTE]
> Use el mismo espacio de nombres aquí que en los comandos `oc adm policy add-scc-to-user` anteriores. Un ejemplo es `arc`.

```console
azdata arc dc create --profile-name azure-arc-azure-openshift --namespace arc --name arc --subscription <subscription id> --resource-group <resource group name> --location <location> --connectivity-mode indirect

#Example
#azdata arc dc create --profile-name azure-arc-azure-openshift --namespace arc --name arc --subscription 1e5ff510-76cf-44cc-9820-82f2d9b51951 --resource-group my-resource-group --location eastus --connectivity-mode indirect
```

Una vez ejecutado el comando, continúe en [Supervisión del estado de creación](#monitoring-the-creation-status).

### <a name="create-on-red-hat-openshift-container-platform-ocp"></a>Creación en Red Hat OpenShift Container Platform (OCP)

> [!NOTE]
> Si usa Red Hat OpenShift Container Platform en Azure, se recomienda usar la versión más reciente disponible.

Antes de crear el controlador de datos en Red Hat OCP, deberá aplicar restricciones de contexto de seguridad específicas. 

#### <a name="apply-the-security-context-constraint"></a>Aplicación de la restricción de contexto de seguridad

[!INCLUDE [apply-security-context-constraint](includes/apply-security-context-constraint.md)]

#### <a name="determine-storage-class"></a>Determinación de la clase de almacenamiento

También necesitará determinar qué clase de almacenamiento se va a usar ejecutando el siguiente comando.

```console
kubectl get storageclass
```

#### <a name="create-custom-deployment-profile"></a>Creación de un perfil de implementación personalizado

Cree un nuevo archivo de perfil de implementación personalizado basado en el perfil de implementación `azure-arc-openshift` ejecutando el siguiente comando. Este comando crea un directorio `custom` en el directorio de trabajo actual y un archivo de perfil de implementación personalizado `control.json` en ese directorio.

Use el perfil `azure-arc-openshift` para OpenShift Container Platform.

```console
azdata arc dc config init --source azure-arc-openshift --path ./custom
```

#### <a name="set-storage-class"></a>Establecimiento de la clase de almacenamiento 

Ahora, establezca la clase de almacenamiento deseada reemplazando `<storageclassname>` en el siguiente comando por el nombre de la clase de almacenamiento que desea usar y que se determinó mediante la ejecución del comando `kubectl get storageclass` anterior.

```console
azdata arc dc config replace --path ./custom/control.json --json-values "spec.storage.data.className=<storageclassname>"
azdata arc dc config replace --path ./custom/control.json --json-values "spec.storage.logs.className=<storageclassname>"

#Example:
#azdata arc dc config replace --path ./custom/control.json --json-values "spec.storage.data.className=mystorageclass"
#azdata arc dc config replace --path ./custom/control.json --json-values "spec.storage.logs.className=mystorageclass"
```

#### <a name="set-loadbalancer-optional"></a>Establecimiento de LoadBalancer (opcional)

De forma predeterminada, el perfil de implementación `azure-arc-openshift` usa `NodePort` como el tipo de servicio. Si usa un clúster de OpenShift que está integrado con un equilibrador de carga, puede cambiar la configuración para usar el tipo de servicio `LoadBalancer` con el siguiente comando:

```console
azdata arc dc config replace --path ./custom/control.json --json-values "$.spec.services[*].serviceType=LoadBalancer"
```

#### <a name="verify-security-policies"></a>Directivas de comprobación de seguridad

Cuando usa OpenShift, es posible que desee realizar la ejecución con las directivas de seguridad predeterminadas de OpenShift o que desee bloquear el entorno más de lo habitual. Si lo desea, puede optar por deshabilitar, mediante el uso de los siguientes comandos, algunas características para reducir los permisos que se necesitan al implementar o al ejecutar.

Este comando deshabilita la recopilación de métricas sobre los pods. Si deshabilita esta característica, no podrá ver las métricas sobre los pods en los paneles de Grafana. El valor predeterminado es true.

```console
azdata arc dc config replace -p ./custom/control.json --json-values spec.security.allowPodMetricsCollection=false
```

Este comando deshabilita la recopilación de métricas sobre los nodos. Si deshabilita esta característica, no podrá ver las métricas sobre los nodos en los paneles de Grafana. El valor predeterminado es true.

```console
azdata arc dc config replace --path ./custom/control.json --json-values spec.security.allowNodeMetricsCollection=false
```

Este comando deshabilita la posibilidad de realizar volcados de memoria para solucionar problemas.
```console
azdata arc dc config replace --path ./custom/control.json --json-values spec.security.allowDumps=false
```

#### <a name="create-data-controller"></a>Creación de un controlador de datos

Ahora ya puede crear el controlador de datos con el siguiente comando.

> [!NOTE]
>   Use el mismo espacio de nombres aquí que en los comandos `oc adm policy add-scc-to-user` anteriores. Un ejemplo es `arc`.

> [!NOTE]
>   El parámetro `--path` debe apuntar al _directorio_ que contiene el archivo control.json y no al propio archivo control.json.


```console
azdata arc dc create --path ./custom --namespace arc --name arc --subscription <subscription id> --resource-group <resource group name> --location <location> --connectivity-mode indirect

#Example:
#azdata arc dc create --path ./custom --namespace arc --name arc --subscription 1e5ff510-76cf-44cc-9820-82f2d9b51951 --resource-group my-resource-group --location eastus --connectivity-mode indirect
```

Una vez ejecutado el comando, continúe en [Supervisión del estado de creación](#monitoring-the-creation-status).

### <a name="create-on-open-source-upstream-kubernetes-kubeadm"></a>Creación en Kubernetes ascendente de código abierto (kubeadm)

De forma predeterminada, el perfil de implementación de kubeadm usa una clase de almacenamiento denominada `local-storage` y el tipo de servicio `NodePort`. Si le resulta aceptable, puede omitir las instrucciones que se indican a continuación para establecer la clase de almacenamiento y el tipo de servicio que desee y ejecutar inmediatamente el comando `azdata arc dc create` siguiente.

Si desea personalizar el perfil de implementación para especificar una clase de almacenamiento o un tipo de servicio específicos, empiece por crear un nuevo archivo de perfil de implementación personalizado basado en el perfil de implementación de kubeadm. Para ello, ejecute el siguiente comando. Este comando creará un directorio `custom` en el directorio de trabajo actual y un archivo de perfil de implementación personalizado `control.json` en ese directorio.

```console
azdata arc dc config init --source azure-arc-kubeadm --path ./custom
```

Puede buscar las clases de almacenamiento disponibles mediante la ejecución del siguiente comando.

```console
kubectl get storageclass
```

Ahora, establezca la clase de almacenamiento deseada reemplazando `<storageclassname>` en el siguiente comando por el nombre de la clase de almacenamiento que desea usar y que se determinó mediante la ejecución del comando `kubectl get storageclass` anterior.

```console
azdata arc dc config replace --path ./custom/control.json --json-values "spec.storage.data.className=<storageclassname>"
azdata arc dc config replace --path ./custom/control.json --json-values "spec.storage.logs.className=<storageclassname>"

#Example:
#azdata arc dc config replace --path ./custom/control.json --json-values "spec.storage.data.className=mystorageclass"
#azdata arc dc config replace --path ./custom/control.json --json-values "spec.storage.logs.className=mystorageclass"
```

De forma predeterminada, el perfil de implementación de kubeadm usa `NodePort` como tipo de servicio. Si usa un clúster de Kubernetes que está integrado con un equilibrador de carga, puede cambiar la configuración con el siguiente comando.

```console
azdata arc dc config replace --path ./custom/control.json --json-values "$.spec.services[*].serviceType=LoadBalancer"
```

Ahora ya puede crear el controlador de datos con el siguiente comando.

```console
azdata arc dc create --path ./custom --namespace arc --name arc --subscription <subscription id> --resource-group <resource group name> --location <location> --connectivity-mode indirect

#Example:
#azdata arc dc create --path ./custom --namespace arc --name arc --subscription 1e5ff510-76cf-44cc-9820-82f2d9b51951 --resource-group my-resource-group --location eastus --connectivity-mode indirect
```

Una vez ejecutado el comando, continúe en [Supervisión del estado de creación](#monitoring-the-creation-status).

### <a name="create-on-aws-elastic-kubernetes-service-eks"></a>Creación en Elastic Kubernetes Service (EKS) de AWS

De forma predeterminada, la clase de almacenamiento de EKS es `gp2` y el tipo de servicio es `LoadBalancer`.

Ejecute el siguiente comando para crear el controlador de datos mediante el perfil de implementación de EKS proporcionado.

```console
azdata arc dc create --profile-name azure-arc-eks --namespace arc --name arc --subscription <subscription id> --resource-group <resource group name> --location <location> --connectivity-mode indirect

#Example:
#azdata arc dc create --profile-name azure-arc-eks --namespace arc --name arc --subscription 1e5ff510-76cf-44cc-9820-82f2d9b51951 --resource-group my-resource-group --location eastus --connectivity-mode indirect
```

Una vez ejecutado el comando, continúe en [Supervisión del estado de creación](#monitoring-the-creation-status).

### <a name="create-on-google-cloud-kubernetes-engine-service-gke"></a>Creación en Kubernetes Engine Service (GKE) de Google Cloud

De forma predeterminada, la clase de almacenamiento de GKE es `standard` y el tipo de servicio es `LoadBalancer`.

Ejecute el siguiente comando para crear el controlador de datos mediante el perfil de implementación de GKE proporcionado.

```console
azdata arc dc create --profile-name azure-arc-gke --namespace arc --name arc --subscription <subscription id> --resource-group <resource group name> --location <location> --connectivity-mode indirect

#Example:
#azdata arc dc create --profile-name azure-arc-gke --namespace arc --name arc --subscription 1e5ff510-76cf-44cc-9820-82f2d9b51951 --resource-group my-resource-group --location eastus --connectivity-mode indirect
```

Una vez ejecutado el comando, continúe en [Supervisión del estado de creación](#monitoring-the-creation-status).

## <a name="monitoring-the-creation-status"></a>Supervisión del estado de creación

La creación del controlador tardará unos minutos en completarse. Puede supervisar el progreso en otra ventana de terminal con los siguientes comandos:

> [!NOTE]
>  En los siguientes comandos de ejemplo se da por hecho que ha creado un controlador de datos y un espacio de nombres de Kubernetes con el nombre `arc`. Si ha usado otro nombre de controlador de datos o de espacio de nombres, puede reemplazar `arc` por ese nombre.

```console
kubectl get datacontroller/arc --namespace arc
```

```console
kubectl get pods --namespace arc
```

También puede comprobar el estado de creación de un pod determinado ejecutando un comando como el siguiente. Esto es especialmente útil para solucionar problemas.

```console
kubectl describe po/<pod name> --namespace arc

#Example:
#kubectl describe po/control-2g7bl --namespace arc
```

## <a name="troubleshooting-creation-problems"></a>Solución de problemas de creación

Si tiene problemas con la creación, consulte la [guía de solución de problemas](troubleshoot-guide.md).