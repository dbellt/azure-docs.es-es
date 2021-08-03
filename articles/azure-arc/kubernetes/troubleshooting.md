---
title: Solución de problemas comunes de Kubernetes habilitado para Azure Arc
services: azure-arc
ms.service: azure-arc
ms.date: 05/21/2021
ms.topic: article
author: mlearned
ms.author: mlearned
description: Solución de problemas comunes con los clústeres de Kubernetes habilitado para Arc
keywords: Kubernetes, Arc, Azure, containers
ms.openlocfilehash: c05e82b084e49958a8c99bc755bdf954b708d69e
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "110795088"
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

La advertencia anterior se observa cuando se ha usado una entidad de servicio para iniciar sesión en Azure y esta entidad de servicio no tiene permisos para obtener información de la aplicación usada por el servicio Azure Arc. Para evitar este error, ejecute los siguientes pasos:

1. Capture el identificador de objeto de la aplicación de Azure AD que utiliza el servicio Azure Arc:

    ```console
    az ad sp show --id 'bc313c14-388c-4e7d-a58e-70017303ee3b' --query objectId -o tsv
    ```

1. Use el valor `<objectId>` del paso anterior para habilitar la característica de ubicaciones personalizadas en el clúster:
    - Si va a habilitar la característica de ubicaciones personalizadas como parte de la conexión del clúster a Arc, ejecute el siguiente comando:

        ```console
        az connectedk8s connect -n <cluster-name> -g <resource-group-name> --custom-locations-oid <objectId>   
        ```

    - Si va a habilitar la característica de ubicaciones personalizadas en un clúster de Kubernetes existente habilitado para Arc, ejecute el siguiente comando:

        ```console
        az connectedk8s enable-features -n <cluster-name> -g <resource-group-name> --custom-locations-oid <objectId> --features cluster-connect custom-locations
        ```

Una vez que se concedan los permisos anteriores, podrá continuar con la [habilitación de la característica de ubicación personalizada](custom-locations.md#enable-custom-locations-on-cluster) en el clúster.

## <a name="arc-enabled-open-service-mesh"></a>Open Service Mesh habilitado para Arc

Los siguientes pasos de solución de problemas proporcionan instrucciones sobre la validación de la implementación de todos los componentes de la extensión Open Service Mesh en el clúster.

### <a name="1-check-osm-controller-deployment"></a>1. Comprobación de la **implementación** del controlador de OSM
```bash
kubectl get deployment -n arc-osm-system --selector app=osm-controller
```

Si el controlador de OSM es correcto, obtendrá una salida parecida a la siguiente:
```
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
osm-controller   1/1     1            1           59m
```

### <a name="2-check-the-osm-controller-pod"></a>2. Comprobación del **pod** del controlador de OSM
```bash
kubectl get pods -n arc-osm-system --selector app=osm-controller
```

Si el controlador de OSM es correcto, obtendrá una salida parecida a la siguiente:
```
NAME                            READY   STATUS    RESTARTS   AGE
osm-controller-b5bd66db-wglzl   0/1     Evicted   0          61m
osm-controller-b5bd66db-wvl9w   1/1     Running   0          31m
```

Aunque en algún momento se _expulsó_ un controlador, hay otro con el estado `READY 1/1` y `Running` con `0` reinicios.
Si el valor de la columna `READY` es distinto de `1/1`, la malla del servicio estaría en un estado interrumpido.
La columna `READY` con el valor `0/1` indica que el contenedor del plano de control se está bloqueando: necesitamos obtener registros. Consulte `Get OSM Controller Logs from Azure Support Center` en la sección siguiente.
Una columna `READY` con un número mayor que 1 después de `/` indicaría que hay archivos de tipo sidecar instalados. Lo más probable es que el controlador de OSM no funcione con los sidecars conectados a él.

### <a name="3-check-osm-controller-service"></a>3. Comprobación del **servicio** de controlador de OSM
```bash
kubectl get service -n arc-osm-system osm-controller
```

Si el controlador de OSM es correcto, tendrá la siguiente salida:
```
NAME             TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)              AGE
osm-controller   ClusterIP   10.0.31.254   <none>        15128/TCP,9092/TCP   67m
```

> Nota: el `CLUSTER-IP` sería diferente. Los valores `NAME` y `PORT(S)` del servicio deben ser los mismos que aparecen en la salida.

### <a name="4-check-osm-controller-endpoints"></a>4. Comprobación de los **puntos de conexión** del controlador de OSM:
```bash
kubectl get endpoints -n arc-osm-system osm-controller
```

Si el controlador de OSM es correcto, obtendrá una salida parecida a la siguiente:
```
NAME             ENDPOINTS                              AGE
osm-controller   10.240.1.115:9092,10.240.1.115:15128   69m
```

Si el clúster del usuario no tiene `ENDPOINTS` para `osm-controller`, eso indica que el plano de control es incorrecto. Esto puede deberse a que el pod del controlador de OSM se bloquea o que no se implementó correctamente.

### <a name="5-check-osm-injector-deployment"></a>5. Comprobación de la **implementación** del inyector de OSM
```bash
kubectl get deployments -n arc-osm-system osm-injector
```

Si el inyector de OSM es correcto, obtendrá una salida parecida a la siguiente:
```
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
osm-injector   1/1     1            1           73m
```

### <a name="6-check-osm-injector-pod"></a>6. Comprobación del **pod** del inyector de OSM
```bash
kubectl get pod -n arc-osm-system --selector app=osm-injector
```

Si el inyector de OSM es correcto, obtendrá una salida parecida a la siguiente:
```
NAME                            READY   STATUS    RESTARTS   AGE
osm-injector-5986c57765-vlsdk   1/1     Running   0          73m
```

La columna `READY` debe ser `1/1`. Cualquier otro valor indicaría un pod osm-injector incorrecto.

### <a name="7-check-osm-injector-service"></a>7. Comprobación del **servicio** del inyector de OSM
```bash
kubectl get service -n arc-osm-system osm-injector
```

Si el inyector de OSM es correcto, obtendrá una salida parecida a la siguiente:
```
NAME           TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
osm-injector   ClusterIP   10.0.39.54   <none>        9090/TCP   75m
```

Asegúrese de que la dirección IP que aparece para el servicio `osm-injector` es `9090`. No debería ser una `EXTERNAL-IP`.

### <a name="8-check-osm-injector-endpoints"></a>8. Comprobación de los **puntos de conexión** del inyector de OSM
```bash
kubectl get endpoints -n arc-osm-system osm-injector
```

Si el inyector de OSM es correcto, obtendrá una salida parecida a la siguiente:
```
NAME           ENDPOINTS           AGE
osm-injector   10.240.1.172:9090   75m
```

Para que OSM funcione, debe haber al menos un punto de conexión para `osm-injector`. La dirección IP de los puntos de conexión del inyector de OSM será diferente. El puerto `9090` debe ser el mismo.


### <a name="9-check-validating-and-mutating-webhooks"></a>9. Comprobación de los webhooks de **validación** y **mutación**
```bash
kubectl get ValidatingWebhookConfiguration --selector app=osm-controller
```

Si el webhook de validación es correcto, obtendrá una salida parecida a la siguiente:
```
NAME              WEBHOOKS   AGE
arc-osm-webhook-osm   1      81m
```

```bash
kubectl get MutatingWebhookConfiguration --selector app=osm-controller
```


Si el webhook de mutación es correcto, obtendrá una salida parecida a la siguiente:
```
NAME              WEBHOOKS   AGE
arc-osm-webhook-osm   1      102m
```

Comprobación del servicio y el paquete de CA del webhook de **validación**:
```
kubectl get ValidatingWebhookConfiguration arc-osm-webhook-osm -o json | jq '.webhooks[0].clientConfig.service'
```

Una configuración del webhook de validación bien realizada tendría la siguiente salida:
```json
{
  "name": "osm-config-validator",
  "namespace": "arc-osm-system",
  "path": "/validate-webhook",
  "port": 9093
}
```

Comprobación del servicio y el paquete de CA del webhook de **mutación**:
```bash
kubectl get MutatingWebhookConfiguration arc-osm-webhook-osm -o json | jq '.webhooks[0].clientConfig.service'
```

Una configuración del webhook de mutación bien realizada tendría la siguiente salida:
```
{
  "name": "osm-injector",
  "namespace": "arc-osm-system",
  "path": "/mutate-pod-creation",
  "port": 9090
}
```


Compruebe si el controlador de OSM ha dado al webhook de validación (o de mutación) un paquete de CA mediante el comando siguiente:

```bash
kubectl get ValidatingWebhookConfiguration arc-osm-webhook-osm -o json | jq -r '.webhooks[0].clientConfig.caBundle' | wc -c
```

```bash
kubectl get MutatingWebhookConfiguration arc-osm-webhook-osm -o json | jq -r '.webhooks[0].clientConfig.caBundle' | wc -c
```

Salida de ejemplo:
```bash
1845
```
El número de la salida indica el número de bytes o el tamaño del paquete de CA. Si está vacío o es 0 o algún número por debajo de 1000, el paquete de CA no está correctamente aprovisionado. Sin un paquete de CA correcto, ValidatingWebhook produciría un error y le impediría realizar cambios en ConfigMap de `osm-config` en el espacio de nombres `arc-osm-system`.

Observemos un ejemplo de error en el que el paquete de CA es incorrecto:
- Un intento de cambiar el valor ConfigMap de `osm-config`:
  ```bash
  kubectl patch ConfigMap osm-config -n arc-osm-system --type merge --patch '{"data":{"config_resync_interval":"2m"}}'
  ```
- Salida del error:
  ```bash
  Error from server (InternalError): Internal error occurred: failed calling webhook "osm-config-webhook.k8s.io": Post https://osm-config-validator.arc-osm-system.svc:9093/validate-webhook?timeout=30s: x509: certificate signed by unknown authority
  ```

Use una de las siguientes soluciones alternativas cuando la configuración del webhook de **validación** tenga un certificado no válido:
- Opción 1. Reinicio del controlador de OSM: se reiniciará el controlador de OSM. En el inicio, sobrescribirá el paquete de CA de los webhooks de mutación y validación.
  ```bash
  kubectl rollout restart deployment -n arc-osm-system osm-controller
  ```

- Opción 2. Eliminación del webhook de validación: al quitar el webhook de validación, las mutaciones del ConfigMap `osm-config` ya no se validan. Cualquier revisión se realizará. Es posible que sea necesario reiniciar el controlador de OSM para volver a escribir rápidamente el paquete de CA.
   ```bash
   kubectl delete ValidatingWebhookConfiguration arc-osm-webhook-osm
   ```

- Opción 3. Eliminación y revisión: el siguiente comando eliminará el webhook de validación, lo que le permitirá agregar valores, e intentará aplicar una revisión de inmediato.
  ```bash
  kubectl delete ValidatingWebhookConfiguration arc-osm-webhook-osm; kubectl patch ConfigMap osm-config -n arc-osm-system --type merge --patch '{"data":{"config_resync_interval":"15s"}}'
  ```


### <a name="10-check-the-osm-config-configmap"></a>10. Comprobación del **ConfigMap** de `osm-config`

>[!Note]
>El controlador de OSM no requiere que el ConfigMap de `osm-config` esté presente en el espacio de nombres `arc-osm-system`. El controlador tiene valores predeterminados razonables para la configuración y puede funcionar sin él.

Comprobación de la existencia:
```bash
kubectl get ConfigMap -n arc-osm-system osm-config
```

Comprobación del contenido del ConfigMap de `osm-config`:
```bash
kubectl get ConfigMap -n arc-osm-system osm-config -o json | jq '.data'     
```
Obtendrá la salida siguiente:
```json
{
  "egress": "false",
  "enable_debug_server": "false",
  "enable_privileged_init_container": "false",
  "envoy_log_level": "error",
  "permissive_traffic_policy_mode": "true",
  "prometheus_scraping": "true",
  "service_cert_validity_duration": "24h",
  "tracing_enable": "false",
  "use_https_ingress": "false",
}
```

Consulte la [documentación de ConfigMap de OSM](https://release-v0-8.docs.openservicemesh.io/docs/osm_config_map/) para comprender los valores de ConfigMap de `osm-config`.

### <a name="11-check-namespaces"></a>11. Comprobación de espacios de nombres

>[!Note]
>El espacio de nombres arc-osm-system nunca participará en una malla de servicio y nunca se etiquetará ni anotará con la clave o los valores siguientes.

Usamos el comando `osm namespace add` para unir espacios de nombres a una malla de servicio determinada.
Cuando un espacio de nombres de Kubernetes forma parte de la malla debe cumplirse lo siguiente:

Vea las anotaciones del espacio de nombres `bookbuyer`:
```bash
kc get namespace bookbuyer -o json | jq '.metadata.annotations'
```

Las siguientes anotaciones deben estar presentes:
```
{
  "openservicemesh.io/sidecar-injection": "enabled"
}
```


Vea las etiquetas del espacio de nombres `bookbuyer`:
```bash
kc get namespace bookbuyer -o json | jq '.metadata.labels'
```

La siguiente etiqueta debe estar presente:
```
{
  "openservicemesh.io/monitored-by": "osm"
}
```
Tenga en cuenta que si no usa la CLI de `osm`, también puede agregar manualmente estas anotaciones a los espacios de nombres. Si un espacio de nombres no está anotado con `"openservicemesh.io/sidecar-injection": "enabled"` o no está etiquetado con `"openservicemesh.io/monitored-by": "osm"`, el inyector de agregará sidecars de Envoy.

>[!Note]
>Después de que se llame a `osm namespace add`, solo se insertarán los pods **nuevos** con un sidecar de Envoy. Los pods existentes deben reiniciarse con el comando `kubectl rollout restard deployment`.


### <a name="12-verify-the-smi-crds"></a>12. Comprobación de las CRD de SMI
Compruebe si el clúster tiene las CRD necesarias:
```bash
kubectl get crds
```

Asegúrese de que las CRD corresponden a la misma versión de nivel superior de OSM. Por ejemplo, Si usa v0.8.4, asegúrese de que las CRD coinciden con las que están disponibles en la rama de la versión v0.8.4 del [proyecto de software de código abierto de OSM](https://docs.openservicemesh.io/). Consulte las [notas de la versión de OSM](https://github.com/openservicemesh/osm/releases).

Obtenga las versiones de las CRD instaladas con el siguiente comando:
```bash
for x in $(kubectl get crds --no-headers | awk '{print $1}' | grep 'smi-spec.io'); do
    kubectl get crd $x -o json | jq -r '(.metadata.name, "----" , .spec.versions[].name, "\n")'
done
```

Si faltan las CRD, use los siguientes comandos para instalarlas en el clúster. Asegúrese de reemplazar la versión en el comando.
```bash
kubectl apply -f https://raw.githubusercontent.com/openservicemesh/osm/release-v0.8.2/charts/osm/crds/access.yaml

kubectl apply -f https://raw.githubusercontent.com/openservicemesh/osm/release-v0.8.2/charts/osm/crds/specs.yaml

kubectl apply -f https://raw.githubusercontent.com/openservicemesh/osm/release-v0.8.2/charts/osm/crds/split.yaml
```
