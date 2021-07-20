---
title: 'Inicio rápido: Implementación de un clúster de AKS mediante la CLI de Azure'
description: Aprenda a crear rápidamente un clúster de Kubernetes, implementar una aplicación y supervisar el rendimiento en Azure Kubernetes Service (AKS) mediante la CLI de Azure.
services: container-service
ms.topic: quickstart
ms.date: 02/26/2021
ms.custom:
- H1Hack27Feb2017
- mvc
- devcenter
- seo-javascript-september2019
- seo-javascript-october2019
- seo-python-october2019
- devx-track-azurecli
- contperf-fy21q1
ms.openlocfilehash: 9814037b9a4ace0a5f39d6ab0b01d1d940fb00fa
ms.sourcegitcommit: 0beea0b1d8475672456da0b3a4485d133283c5ea
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/28/2021
ms.locfileid: "112991989"
---
# <a name="quickstart-deploy-an-azure-kubernetes-service-cluster-using-the-azure-cli"></a>Inicio rápido: Implementación de un clúster de Azure Kubernetes Service mediante la CLI de Azure

Azure Kubernetes Service (AKS) es un servicio de Kubernetes administrado que le permite implementar y administrar clústeres rápidamente. En este inicio rápido realizará lo siguiente:
* Implementación de un clúster de AKS con la CLI de Azure. 
* Ejecución de una aplicación de varios contenedores con un servidor front-end web y una instancia de Redis en el clúster. 
* Supervisión del estado del clúster y de los pods que ejecutan la aplicación.

  ![Aplicación de votación implementada en Azure Kubernetes Service](./media/container-service-kubernetes-walkthrough/voting-app-deployed-in-azure-kubernetes-service.png)

En esta guía rápida se presupone un conocimiento básico de los conceptos de Kubernetes. Para más información, consulte [Conceptos básicos de Kubernetes de Azure Kubernetes Service (AKS)][kubernetes-concepts].

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

Para más información sobre cómo crear un grupo de nodos de Windows Server, consulte [Creación de un clúster de AKS que admita contenedores de Windows Server][windows-container-cli].

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- En este artículo se necesita la versión 2.0.64, o versiones posteriores, de la CLI de Azure. Si usa Azure Cloud Shell, ya está instalada la versión más reciente.
- La identidad que usa para crear el clúster tiene los permisos mínimos adecuados. Para más información sobre el acceso y la identidad en AKS, consulte [Opciones de acceso e identidad en Azure Kubernetes Service (AKS)](concepts-identity.md).

> [!NOTE]
> Si planea ejecutar los comandos de este inicio rápido localmente en lugar de en Azure Cloud Shell, hágalo como administrador.

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Un [grupo de recursos de Azure](../azure-resource-manager/management/overview.md) es un grupo lógico en el que se implementan y administran recursos de Azure. Cuando crea un grupo de recursos, se le pide que especifique una ubicación. Esta ubicación es: 
* La ubicación de almacenamiento de los metadatos del grupo de recursos.
* El lugar en el que se ejecutan los recursos en Azure si no se especifica otra región al crearlos. 

En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroup* en la ubicación *eastus*.

Cree un grupo de recursos con el comando [az group create][az-group-create].


```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

Salida del grupo de recursos que se creó correctamente:

```json
{
  "id": "/subscriptions/<guid>/resourceGroups/myResourceGroup",
  "location": "eastus",
  "managedBy": null,
  "name": "myResourceGroup",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null
}
```

## <a name="enable-cluster-monitoring"></a>Habilitación de la supervisión de clústeres

1. Compruebe que *Microsoft.OperationsManagement* y *Microsoft.OperationalInsights* están registrados en la suscripción. Para comprobar el estado del registro, siga estos pasos:

    ```azurecli
    az provider show -n Microsoft.OperationsManagement -o table
    az provider show -n Microsoft.OperationalInsights -o table
    ```
 
    Si no están registrados, use el siguiente comando para registrar *Microsoft.OperationsManagement* y *Microsoft.OperationalInsights*:
 
    ```azurecli
    az provider register --namespace Microsoft.OperationsManagement
    az provider register --namespace Microsoft.OperationalInsights
    ```

2. Habilite [Azure Monitor para contenedores][azure-monitor-containers] mediante el parámetro *--enable-addons monitoring*. 

## <a name="create-aks-cluster"></a>Creación de un clúster de AKS

Cree un clúster de AKS con el comando [az aks create][az-aks-create]. En el siguiente ejemplo se crea un clúster denominado *myAKSCluster* con un nodo: 

```azurecli-interactive
az aks create --resource-group myResourceGroup --name myAKSCluster --node-count 1 --enable-addons monitoring --generate-ssh-keys
```

Transcurridos unos minutos, el comando se completa y devuelve información en formato JSON sobre el clúster.

> [!NOTE]
> Al crear un clúster de AKS, se crea automáticamente un segundo grupo de recursos para almacenar los recursos de dicho clúster. Para más información, consulte [¿Por qué se crean dos grupos de recursos con AKS?](./faq.md#why-are-two-resource-groups-created-with-aks)

## <a name="connect-to-the-cluster"></a>Conectarse al clúster

Para administrar un clúster de Kubernetes, use [kubectl][kubectl], el cliente de línea de comandos de Kubernetes. Si usa Azure Cloud Shell, `kubectl` ya está instalado. 

1. Instale `kubectl` localmente mediante el comando [az aks install-cli][az-aks-install-cli]:

    ```azurecli
    az aks install-cli
    ```

2. Para configurar `kubectl` para conectarse a su clúster de Kubernetes, use el comando [az aks get-credentials][az-aks-get-credentials]. El siguiente comando:  
      * Descarga las credenciales y configura la CLI de Kubernetes para usarlas.
      * Usa `~/.kube/config`, la ubicación predeterminada del [archivo de configuración de Kubernetes][kubeconfig-file]. Puede especificar otra ubicación para el archivo de configuración de Kubernetes con la opción *--file*.


    ```azurecli-interactive
    az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
    ```

3. Compruebe la conexión al clúster con el comando [kubectl get][kubectl-get]. Este comando devuelve una lista de los nodos del clúster.

    ```azurecli-interactive
    kubectl get nodes
    ```

    La salida muestra el nodo único creado en los pasos anteriores. Asegúrese de que el estado del nodo es *Listo*:

    ```output
    NAME                       STATUS   ROLES   AGE     VERSION
    aks-nodepool1-31718369-0   Ready    agent   6m44s   v1.12.8
    ```

## <a name="run-the-application"></a>Ejecución de la aplicación

Un [archivo de manifiesto de Kubernetes][kubernetes-deployment] define el estado deseado del clúster, por ejemplo, qué imágenes de contenedor se van a ejecutar. 

En este inicio rápido se usa un manifiesto para crear todos los objetos necesarios para ejecutar la [aplicación Azure Vote][azure-vote-app]. Este manifiesto incluye dos [implementaciones de Kubernetes][kubernetes-deployment]:
* Las aplicaciones de Python de ejemplo de Azure Vote.
* Una instancia de Redis. 

También se crean dos [servicios de Kubernetes][kubernetes-service]:
* Un servicio interno para la instancia de Redis.
* Un servicio externo para acceder a la aplicación Azure Vote desde Internet.

1. Cree un archivo llamado `azure-vote.yaml`.
    * Si usa Azure Cloud Shell, este archivo se puede crear mediante `code`, `vi` o `nano` como si trabajara en un sistema físico o virtual:
1. Copie la siguiente definición de código YAML:

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: azure-vote-back
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: azure-vote-back
      template:
        metadata:
          labels:
            app: azure-vote-back
        spec:
          nodeSelector:
            "beta.kubernetes.io/os": linux
          containers:
          - name: azure-vote-back
            image: mcr.microsoft.com/oss/bitnami/redis:6.0.8
            env:
            - name: ALLOW_EMPTY_PASSWORD
              value: "yes"
            resources:
              requests:
                cpu: 100m
                memory: 128Mi
              limits:
                cpu: 250m
                memory: 256Mi
            ports:
            - containerPort: 6379
              name: redis
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: azure-vote-back
    spec:
      ports:
      - port: 6379
      selector:
        app: azure-vote-back
    ---
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: azure-vote-front
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: azure-vote-front
      template:
        metadata:
          labels:
            app: azure-vote-front
        spec:
          nodeSelector:
            "beta.kubernetes.io/os": linux
          containers:
          - name: azure-vote-front
            image: mcr.microsoft.com/azuredocs/azure-vote-front:v1
            resources:
              requests:
                cpu: 100m
                memory: 128Mi
              limits:
                cpu: 250m
                memory: 256Mi
            ports:
            - containerPort: 80
            env:
            - name: REDIS
              value: "azure-vote-back"
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: azure-vote-front
    spec:
      type: LoadBalancer
      ports:
      - port: 80
      selector:
        app: azure-vote-front
    ```

1. Implemente la aplicación mediante el comando [kubectl apply][kubectl-apply] y especifique el nombre del manifiesto de YAML:

    ```console
    kubectl apply -f azure-vote.yaml
    ```

    La salida muestra los servicios y las implementaciones que se crearon correctamente:

    ```output
    deployment "azure-vote-back" created
    service "azure-vote-back" created
    deployment "azure-vote-front" created
    service "azure-vote-front" created
    ```

## <a name="test-the-application"></a>Prueba de la aplicación

Cuando se ejecuta la aplicación, un servicio de Kubernetes expone el front-end de la aplicación a Internet. Este proceso puede tardar unos minutos en completarse.

Para supervisar el progreso, utilice el comando [kubectl get service][kubectl-get] con el argumento `--watch`.

```azurecli-interactive
kubectl get service azure-vote-front --watch
```

La salida de **EXTERNAL-IP** del servicio `azure-vote-front` aparecerá inicialmente como *pendiente*.

```output
NAME               TYPE           CLUSTER-IP   EXTERNAL-IP   PORT(S)        AGE
azure-vote-front   LoadBalancer   10.0.37.27   <pending>     80:30572/TCP   6s
```

Una vez que la dirección **EXTERNAL-IP** cambia de *pendiente* a una dirección IP pública real, use `CTRL-C` para detener el `kubectl` proceso de inspección. En la salida del ejemplo siguiente se muestra una dirección IP pública válida asignada al servicio:

```output
azure-vote-front   LoadBalancer   10.0.37.27   52.179.23.131   80:30572/TCP   2m
```

Para ver la aplicación Azure Vote en acción, abra un explorador web en la dirección IP externa del servicio.

![Aplicación de votación implementada en Azure Kubernetes Service](./media/container-service-kubernetes-walkthrough/voting-app-deployed-in-azure-kubernetes-service.png)

Vea las métricas de mantenimiento de los nodos del clúster y los pods capturadas por [Azure Monitor para contenedores][azure-monitor-containers] en Azure Portal. 

## <a name="delete-the-cluster"></a>Eliminación del clúster

Para evitar cargos de Azure, se recomienda limpiar los recursos que no sean necesarios. Use el comando [az group delete][az-group-delete] para quitar el grupo de recursos, el servicio de contenedor y todos los recursos relacionados.

```azurecli-interactive
az group delete --name myResourceGroup --yes --no-wait
```

> [!NOTE]
> Cuando elimina el clúster, la entidad de servicio Azure Active Directory que utiliza el clúster de AKS no se quita. Para conocer los pasos que hay que realizar para quitar la entidad de servicio, consulte [Consideraciones principales y eliminación de AKS][sp-delete].
> 
> Si usó una identidad administrada, esta está administrada por la plataforma y no requiere que la quite.

## <a name="get-the-code"></a>Obtención del código

En este inicio rápido, se han usado imágenes de un contenedor creado previamente para crear una implementación de Kubernetes. El código de la aplicación relacionada, Dockerfile, y el archivo de manifiesto de Kubernetes están [disponibles en GitHub][azure-vote-app].

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha implementado un clúster de Kubernetes y luego ha implementado en él una aplicación de varios contenedores. [Acceda al panel web de Kubernetes][kubernetes-dashboard] del clúster de AKS.

Para obtener más información sobre AKS y un ejemplo completo desde el código hasta la implementación, continúe con el tutorial del clúster de Kubernetes.

> [!div class="nextstepaction"]
> [Tutorial de AKS][aks-tutorial]

<!-- LINKS - external -->
[azure-vote-app]: https://github.com/Azure-Samples/azure-voting-app-redis.git
[kubectl]: https://kubernetes.io/docs/user-guide/kubectl/
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubeconfig-file]: https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/

<!-- LINKS - internal -->
[kubernetes-concepts]: concepts-clusters-workloads.md
[aks-monitor]: ../azure-monitor/containers/container-insights-onboard.md
[aks-tutorial]: ./tutorial-kubernetes-prepare-app.md
[az-aks-browse]: /cli/azure/aks#az_aks_browse
[az-aks-create]: /cli/azure/aks#az_aks_create
[az-aks-get-credentials]: /cli/azure/aks#az_aks_get_credentials
[az-aks-install-cli]: /cli/azure/aks#az_aks_install_cli
[az-group-create]: /cli/azure/group#az_group_create
[az-group-delete]: /cli/azure/group#az_group_delete
[azure-cli-install]: /cli/azure/install_azure_cli
[azure-monitor-containers]: ../azure-monitor/containers/container-insights-overview.md
[sp-delete]: kubernetes-service-principal.md#additional-considerations
[azure-portal]: https://portal.azure.com
[kubernetes-deployment]: concepts-clusters-workloads.md#deployments-and-yaml-manifests
[kubernetes-service]: concepts-network.md#services
[kubernetes-dashboard]: kubernetes-dashboard.md
[windows-container-cli]: windows-container-cli.md
