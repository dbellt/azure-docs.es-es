---
title: Solución de problemas comunes de Kubernetes habilitado para Azure Arc
services: azure-arc
ms.service: azure-arc
ms.date: 03/02/2020
ms.topic: article
author: mlearned
ms.author: mlearned
description: Solución de problemas comunes con los clústeres de Kubernetes habilitado para Arc
keywords: Kubernetes, Arc, Azure, containers
ms.openlocfilehash: f0b02e5b4e58cda246751b16542a0a2ac587e7b6
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108289631"
---
# <a name="azure-arc-enabled-kubernetes-troubleshooting"></a>Solución de problemas de Kubernetes habilitado para Azure Arc

En este documento, se proporcionan algunas guías para solucionar problemas comunes relacionados con la conectividad, los permisos y los agentes.

## <a name="general-troubleshooting"></a>Solución general de problemas

### <a name="azure-cli"></a>Azure CLI

Para poder utilizar los comandos `az connectedk8s` o `az k8s-configuration` de la CLI, asegúrese de que la CLI de Azure se ha configurado para que utilice la suscripción de Azure apropiada.

```azurecli
az account set --subscription 'subscriptionId'
az account show
```

### <a name="azure-arc-agents"></a>Agentes de Azure Arc

Todos los agentes de Kubernetes habilitado para Azure Arc se implementan como pods en el espacio de nombres `azure-arc`. Todos los pods deben ejecutarse y pasar sus comprobaciones de estado.

En primer lugar, compruebe la versión de Helm para Azure Arc:

```console
$ helm --namespace default status azure-arc
NAME: azure-arc
LAST DEPLOYED: Fri Apr  3 11:13:10 2020
NAMESPACE: default
STATUS: deployed
REVISION: 5
TEST SUITE: None
```

Si no se encuentra la versión de Helm o no está presente, intente volver a [conectar el clúster a Azure Arc](./quickstart-connect-cluster.md).

Si la versión de Helm está presente y tiene el estado `STATUS: deployed`, compruebe el estado de los agentes con `kubectl`:

```console
$ kubectl -n azure-arc get deployments,pods
NAME                                       READY  UP-TO-DATE  AVAILABLE  AGE
deployment.apps/clusteridentityoperator     1/1       1          1       16h
deployment.apps/config-agent                1/1       1          1       16h
deployment.apps/cluster-metadata-operator   1/1       1          1       16h
deployment.apps/controller-manager          1/1       1          1       16h
deployment.apps/flux-logs-agent             1/1       1          1       16h
deployment.apps/metrics-agent               1/1       1          1       16h
deployment.apps/resource-sync-agent         1/1       1          1       16h

NAME                                            READY   STATUS  RESTART  AGE
pod/cluster-metadata-operator-7fb54d9986-g785b  2/2     Running  0       16h
pod/clusteridentityoperator-6d6678ffd4-tx8hr    3/3     Running  0       16h
pod/config-agent-544c4669f9-4th92               3/3     Running  0       16h
pod/controller-manager-fddf5c766-ftd96          3/3     Running  0       16h
pod/flux-logs-agent-7c489f57f4-mwqqv            2/2     Running  0       16h
pod/metrics-agent-58b765c8db-n5l7k              2/2     Running  0       16h
pod/resource-sync-agent-5cf85976c7-522p5        3/3     Running  0       16h
```

En todos los pods, el valor de `STATUS` debe ser `Running`, con `3/3` o `2/2` en la columna `READY`. Capture los registros y describa los pods que devuelven `Error` o `CrashLoopBackOff`. Si algún pod está bloqueado en el estado `Pending`, es posible que no haya recursos suficientes en los nodos del clúster. [Escalar verticalmente el clúster](https://kubernetes.io/docs/tasks/administer-cluster/) puede hacer que estos pods pasen al estado `Running`.

## <a name="connecting-kubernetes-clusters-to-azure-arc"></a>Conexión de clústeres de Kubernetes con Azure Arc

Para conectar clústeres a Azure, es necesario tener acceso a una suscripción de Azure y a un clúster de destino con el rol `cluster-admin`. Si no puede establecer conexión con el clúster o los permisos no son suficientes, se producirá un error al conectar el clúster a Azure Arc.

### <a name="insufficient-cluster-permissions"></a>Permisos de clúster insuficientes

Si el archivo kubeconfig proporcionado no tiene permisos suficientes para instalar los agentes de Azure Arc, el comando de la CLI de Azure devolverá un error.

```azurecli
$ az connectedk8s connect --resource-group AzureArc --name AzureArcCluster
Ensure that you have the latest helm version installed before proceeding to avoid unexpected errors.
This operation might take a while...

Error: list: failed to list: secrets is forbidden: User "myuser" cannot list resource "secrets" in API group "" at the cluster scope
```

El usuario que conecte el clúster con Azure Arc debe tener el rol `cluster-admin` asignado en el clúster.

### <a name="installation-timeouts"></a>Tiempos de espera de instalación

Para conectar un clúster de Kubernetes a una versión de Kubernetes habilitada para Azure Arc, es necesario instalar agentes de Azure Arc en el clúster. Si el clúster se ejecuta utilizando una conexión a Internet lenta, la extracción de la imagen del contenedor puede agotar los tiempos de espera de la CLI de Azure.

```azurecli
$ az connectedk8s connect --resource-group AzureArc --name AzureArcCluster
Ensure that you have the latest helm version installed before proceeding to avoid unexpected errors.
This operation might take a while...
```

### <a name="helm-issue"></a>Problema de Helm

La versión `v3.3.0-rc.1` de Helm tiene un [problema](https://github.com/helm/helm/pull/8527) y es que la instalación/actualización de Helm (que se utiliza en la extensión `connectedk8s` de la CLI ) provoca la ejecución de todos los enlaces, lo que genera el siguiente error:

```console
$ az connectedk8s connect -n shasbakstest -g shasbakstest
Ensure that you have the latest helm version installed before proceeding.
This operation might take a while...

Please check if the azure-arc namespace was deployed and run 'kubectl get pods -n azure-arc' to check if all the pods are in running state. A possible cause for pods stuck in pending state could be insufficientresources on the kubernetes cluster to onboard to arc.
ValidationError: Unable to install helm release: Error: customresourcedefinitions.apiextensions.k8s.io "connectedclusters.arc.azure.com" not found
```

Para resolver el problema, siga estos pasos:

1. Elimine el recurso de Kubernetes habilitado para Azure Arc en Azure Portal.
2. Ejecute los comandos siguientes en la máquina:
    
    ```console
    kubectl delete ns azure-arc
    kubectl delete clusterrolebinding azure-arc-operator
    kubectl delete secret sh.helm.release.v1.azure-arc.v1
    ```

3. [Instale una versión estable](https://helm.sh/docs/intro/install/) de Helm 3 en el equipo en lugar de la versión candidata para lanzamiento.
4. Ejecute el comando `az connectedk8s connect` con los valores adecuados para conectar el clúster a Azure Arc.

## <a name="configuration-management"></a>Administración de configuración

### <a name="general"></a>General
Para solucionar problemas relacionados con los recursos de configuración, ejecute comandos az con el parámetro `--debug` especificado.

```console
az provider show -n Microsoft.KubernetesConfiguration --debug
az k8s-configuration create <parameters> --debug
```

### <a name="create-configurations"></a>Creación de configuraciones

Los permisos de escritura del recurso de Kubernetes habilitado para Azure Arc ( `Microsoft.Kubernetes/connectedClusters/Write` ) son necesarios y suficientes para crear configuraciones en ese clúster.

### <a name="configuration-remains-pending"></a>La configuración se mantiene como `Pending`

```console
kubectl -n azure-arc logs -l app.kubernetes.io/component=config-agent -c config-agent
$ k -n pending get gitconfigs.clusterconfig.azure.com  -o yaml
apiVersion: v1
items:
- apiVersion: clusterconfig.azure.com/v1beta1
  kind: GitConfig
  metadata:
    creationTimestamp: "2020-04-13T20:37:25Z"
    generation: 1
    name: pending
    namespace: pending
    resourceVersion: "10088301"
    selfLink: /apis/clusterconfig.azure.com/v1beta1/namespaces/pending/gitconfigs/pending
    uid: d9452407-ff53-4c02-9b5a-51d55e62f704
  spec:
    correlationId: ""
    deleteOperator: false
    enableHelmOperator: false
    giturl: git@github.com:slack/cluster-config.git
    helmOperatorProperties: null
    operatorClientLocation: azurearcfork8s.azurecr.io/arc-preview/fluxctl:0.1.3
    operatorInstanceName: pending
    operatorParams: '"--disable-registry-scanning"'
    operatorScope: cluster
    operatorType: flux
  status:
    configAppliedTime: "2020-04-13T20:38:43.081Z"
    isSyncedWithAzure: true
    lastPolledStatusTime: ""
    message: 'Error: {exit status 1} occurred while doing the operation : {Installing
      the operator} on the config'
    operatorPropertiesHashed: ""
    publicKey: ""
    retryCountPublicKey: 0
    status: Installing the operator
kind: List
metadata:
  resourceVersion: ""
  selfLink: ""
```
## <a name="monitoring"></a>Supervisión

Azure Monitor para contenedores requiere que su DaemonSet se ejecute en modo con privilegiado. Para configurar correctamente un clúster de Canonical Charmed Kubernetes para la supervisión, ejecute el siguiente comando:

```console
juju config kubernetes-worker allow-privileged=true
```

## <a name="enable-custom-locations-using-service-principal"></a>Habilitación de ubicaciones personalizadas mediante la entidad de servicio

Al conectar el clúster a Azure Arc o al habilitar la característica de ubicaciones personalizadas en un clúster existente, puede que vea la advertencia siguiente:

```console
Unable to fetch oid of 'custom-locations' app. Proceeding without enabling the feature. Insufficient privileges to complete the operation.
```

La advertencia anterior se observa cuando se ha usado una entidad de servicio para iniciar sesión en Azure y esta entidad de servicio no tiene permisos para obtener información de la aplicación usada por el servicio Azure Arc. Ejecute los siguientes comandos para conceder los permisos necesarios:

```console
az ad app permission add --id <service-principal-app-id> --api 00000002-0000-0000-c000-000000000000 --api-permissions 3afa6a7d-9b1a-42eb-948e-1650a849e176=Role
az ad app permission admin-consent --id <service-principal-app-id>
```

Una vez que se concedan los permisos anteriores, podrá continuar con la [habilitación de la característica de ubicación personalizada](custom-locations.md#enable-custom-locations-on-cluster) en el clúster.