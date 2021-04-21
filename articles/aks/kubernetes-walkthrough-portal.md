---
title: 'Inicio rápido: Implementación de un clúster de AKS mediante Azure Portal'
titleSuffix: Azure Kubernetes Service
description: Aprenda a crear rápidamente un clúster de Kubernetes, implementar una aplicación y supervisar el rendimiento en Azure Kubernetes Service (AKS) mediante Azure Portal.
services: container-service
ms.topic: quickstart
ms.date: 03/15/2021
ms.custom: mvc, seo-javascript-october2019, contperf-fy21q3
ms.openlocfilehash: 28ba2ffd2007aeb45081cf66b05395a2b8456bf7
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107779712"
---
# <a name="quickstart-deploy-an-azure-kubernetes-service-aks-cluster-using-the-azure-portal"></a>Inicio rápido: Implementación de un clúster de Azure Kubernetes Service (AKS) mediante Azure Portal

Azure Kubernetes Service (AKS) es un servicio de Kubernetes administrado que le permite implementar y administrar clústeres rápidamente. En este inicio rápido realizará lo siguiente:
* Implementación de un clúster de AKS en Azure Portal. 
* Ejecución de una aplicación de varios contenedores con un servidor front-end web y una instancia de Redis en el clúster. 
* Supervisión del estado del clúster y de los pods que ejecutan la aplicación.

![Imagen de cómo llegar a la aplicación de ejemplo Voto de Azure](media/container-service-kubernetes-walkthrough/azure-voting-application.png)

En esta guía rápida se presupone un conocimiento básico de los conceptos de Kubernetes. Para más información, consulte [Conceptos básicos de Kubernetes de Azure Kubernetes Service (AKS)][kubernetes-concepts].

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com).

## <a name="create-an-aks-cluster"></a>Creación de un clúster de AKS

1. En el menú de Azure Portal o en la **página principal**, seleccione **Crear un recurso**.

2. Seleccione **Containers** > **Kubernetes Service**.

3. En la página **Datos básicos**, configure las siguientes opciones:
    - **Detalles del proyecto**: 
        * Seleccione una **suscripción** de Azure.
        * Seleccione o cree un **grupo de recursos** de Azure, como *myResourceGroup*.
    - **Detalles del clúster**: 
        * Escriba un **Nombre del clúster de Kubernetes**, como *myAKSCluster*. 
        * En **Región** y **Versión de Kubernetes**, seleccionelos valores adecuados para el clúster de AKS.
    - **Grupo de nodos principal**: 
        * Seleccione un **tamaño de nodo** de máquina virtual para los nodos de AKS. El tamaño de VM *no* puede cambiarse una vez que se ha implementado un clúster de AKS.
        * Seleccione el número de nodos que se van a implementar en el clúster. En esta guía de inicio rápido, establezca **Número de nodos** en *1*. El número de nodos *puede* ajustarse después de implementar el clúster.
    
    ![Creación de un clúster de AKS: proporcionar información básica](media/kubernetes-walkthrough-portal/create-cluster-basics.png)

4. Seleccione **Siguiente: Grupos de nodos** cuando haya terminado.

5. Mantenga las opciones predeterminadas de los **grupos de nodos**. En la parte inferior de la pantalla, haga clic en **Siguiente: Autenticación**.
    > [!CAUTION]
    > Las entidades de servicio de Azure AD recién creadas pueden tardar varios minutos en propagarse y estar disponibles, lo que provoca errores de "entidad de servicio no encontrada" y de validación en Azure Portal. Si se tropieza con esta situación, visite [nuestro artículo de solución de problemas](troubleshooting.md#received-an-error-saying-my-service-principal-wasnt-found-or-is-invalid-when-i-try-to-create-a-new-cluster) para resolverla.

6. En la página **Autenticación**, configure las siguientes opciones:
    - Cree una identidad de clúster; para ello puede:
        * Dejar el campo **Autenticación** con **System-assinged managed identity** (Identidad administrada por el sistema) o
        * Elegir **Entidad de servicio** para usar una entidad de servicio. 
            * Seleccione *Entidad de servicio predeterminada (nueva)* para crear una entidad de servicio predeterminada.
            * También puede elegir *Configurar la entidad de servicio* para usar una existente. Deberá proporcionar el identificador y el secreto de cliente del SPN de la entidad de servicio existente.
    - Habilite la opción de control de acceso basado en rol de Kubernetes (Kubernetes RBAC) para proporcionar un control más preciso sobre el acceso a los recursos de Kubernetes implementados en el clúster de AKS.

    De forma predeterminada, se usa la red *Básica* y está habilitado Azure Monitor para contenedores. 

7. Haga clic en **Revisar y crear** y, luego, en **Crear** cuando finalice la validación. 


8. El clúster de AKS tarda unos minutos en crearse. Una vez finalizada la implementación, vaya al recurso; puede hacerlo de dos maneras:
    * Haga clic en **Ir al recurso**.
    * Vaya al grupo de recursos del clúster de AKS y seleccione el recurso de AKS. 
        * Por ejemplo, en el panel de clúster siguiente, busque *myResourceGroup* y seleccionar el recurso *myAKSCluster*.

        ![Panel de AKS de ejemplo en Azure Portal](media/kubernetes-walkthrough-portal/aks-portal-dashboard.png)

## <a name="connect-to-the-cluster"></a>Conectarse al clúster

Para administrar un clúster de Kubernetes, use [kubectl][kubectl], el cliente de línea de comandos de Kubernetes. Si usa Azure Cloud Shell, `kubectl` ya está instalado. 

1. Abra Cloud Shell mediante el botón `>_` de la parte superior de Azure Portal.

    ![Apertura de Azure Cloud Shell en el portal](media/kubernetes-walkthrough-portal/aks-cloud-shell.png)

    > [!NOTE]
    > Para realizar estas operaciones en una instalación de shell local:
    > 1. Compruebe que la CLI de Azure está instalada.
    > 2. Conéctese a Azure a través del comando `az login`.

2. Para configurar `kubectl` para conectarse a su clúster de Kubernetes, use el comando [az aks get-credentials][az-aks-get-credentials]. El siguiente comando descarga las credenciales y configura la CLI de Kubernetes para usarlas.

    ```azurecli
    az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
    ```

3. Para comprobar la conexión al clúster, use el comando `kubectl get` para devolver una lista de los nodos del clúster.

    ```console
    kubectl get nodes
    ```

    La salida muestra el nodo único creado en los pasos anteriores. Asegúrese de que el estado del nodo es *Listo*:

    ```output
    NAME                       STATUS    ROLES     AGE       VERSION
    aks-agentpool-14693408-0   Ready     agent     15m       v1.11.5
    ```

## <a name="run-the-application"></a>Ejecución de la aplicación

Un archivo de manifiesto de Kubernetes define el estado deseado de un clúster, por ejemplo, qué imágenes de contenedor se van a ejecutar. 

En este inicio rápido se usa un manifiesto para crear todos los objetos necesarios para ejecutar la aplicación Azure Vote. Este manifiesto incluye dos implementaciones de Kubernetes:
* Las aplicaciones de Python de ejemplo de Azure Vote.
* Una instancia de Redis. 

También se crean dos servicios de Kubernetes:
* Un servicio interno para la instancia de Redis.
* Un servicio externo para acceder a la aplicación Azure Vote desde Internet.

1. En Cloud Shell, use un editor para crear un archivo llamado `azure-vote.yaml`, por ejemplo:
    * `code azure-vote.yaml`
    * `nano azure-vote.yaml` o 
    * `vi azure-vote.yaml`. 

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

1. Implemente la aplicación mediante el comando `kubectl apply` y especifique el nombre del manifiesto de YAML:

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

Para supervisar el progreso, utilice el comando `kubectl get service` con el argumento `--watch`.

```console
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

![Imagen de cómo llegar a la aplicación de ejemplo Voto de Azure](media/container-service-kubernetes-walkthrough/azure-voting-application.png)

## <a name="monitor-health-and-logs"></a>Supervisión de estado y registros

Cuando se creó el clúster, se habilitó Azure Monitor para contenedores. Azure Monitor para contenedores proporciona métricas de estado para el clúster de AKS y los pods que se ejecutan en él.

Los datos de métricas tardan unos minutos en rellenarse en Azure Portal. Para ver el estado de mantenimiento, el tiempo de actividad y el uso de recursos actuales de los pods de Azure Vote, haga lo siguiente:

1. Vuelva al recurso de AKS en Azure Portal.
1. En **Supervisión** en el lado izquierdo, elija **Detalles**.
1. En la parte superior, elija **+ Agregar filtro**.
1. Seleccione **Espacio de nombres** como propiedad y, después, elija *\<All but kube-system\>* .
1. Seleccione **Contenedores** para verlos.

Se muestran los contenedores `azure-vote-back` y `azure-vote-front`, como se ilustra en el ejemplo siguiente:

![Visualización del estado de ejecución de contenedores en AKS](media/kubernetes-walkthrough-portal/monitor-containers.png)

Para ver los registros del pod `azure-vote-front`, seleccione el vínculo **Ver registros del contenedor** en el menú desplegable de la lista de contenedores. Estos registros incluyen los flujos *stdout* y *stderr* del contenedor.

![Visualización de los registros de contenedores en AKS](media/kubernetes-walkthrough-portal/monitor-container-logs.png)

## <a name="delete-cluster"></a>Eliminación de clúster

Para evitar cargos de Azure, se recomienda limpiar los recursos que no sean necesarios. Seleccione el botón **Eliminar** en el panel del clúster de AKS. También puede usar el comando [az aks delete][az-aks-delete] en Cloud Shell:

```azurecli
az aks delete --resource-group myResourceGroup --name myAKSCluster --no-wait
```
> [!NOTE]
> Cuando elimina el clúster, la entidad de servicio Azure Active Directory que utiliza el clúster de AKS no se quita. Para conocer los pasos que hay que realizar para quitar la entidad de servicio, consulte [Consideraciones principales y eliminación de AKS][sp-delete].
> 
> Si usó una identidad administrada, esta está administrada por la plataforma y no requiere que la quite.

## <a name="get-the-code"></a>Obtención del código

En este inicio rápido, se han usado imágenes de un contenedor creado previamente para crear una implementación de Kubernetes. El código de la aplicación relacionada, Dockerfile, y el archivo de manifiesto de Kubernetes están [disponibles en GitHub][azure-vote-app].

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha implementado un clúster de Kubernetes y luego ha implementado en él una aplicación de varios contenedores. Acceda al panel web de Kubernetes del clúster de AKS.


Para más información sobre AKS con un recorrido por un ejemplo completo, con la creación de una aplicación, la implementación desde Azure Container Registry, la actualización de una aplicación en ejecución y el escalado y la actualización del clúster, continúe con el tutorial de clústeres de Kubernetes.

> [!div class="nextstepaction"]
> [Tutorial de AKS][aks-tutorial]

<!-- LINKS - external -->
[azure-vote-app]: https://github.com/Azure-Samples/azure-voting-app-redis.git
[kubectl]: https://kubernetes.io/docs/user-guide/kubectl/
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubernetes-documentation]: https://kubernetes.io/docs/home/

<!-- LINKS - internal -->
[kubernetes-concepts]: concepts-clusters-workloads.md
[az-aks-get-credentials]: /cli/azure/aks#az_aks_get_credentials
[az-aks-delete]: /cli/azure/aks#az_aks_delete
[aks-monitor]: ../azure-monitor/containers/container-insights-overview.md
[aks-network]: ./concepts-network.md
[aks-tutorial]: ./tutorial-kubernetes-prepare-app.md
[http-routing]: ./http-application-routing.md
[sp-delete]: kubernetes-service-principal.md#additional-considerations