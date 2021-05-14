---
title: 'Inicio rápido: Creación de un clúster de Azure Kubernetes Service (AKS)'
description: Aprenda a crear rápidamente un clúster de Kubernetes con una plantilla de Azure Resource Manager y a implementar una aplicación en Azure Kubernetes Service (AKS).
services: container-service
ms.topic: quickstart
ms.date: 03/15/2021
ms.custom: mvc,subject-armqs, devx-track-azurecli
ms.openlocfilehash: ea66651818f92dec05de4830726c999d75e5196b
ms.sourcegitcommit: 43be2ce9bf6d1186795609c99b6b8f6bb4676f47
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2021
ms.locfileid: "108279719"
---
# <a name="quickstart-deploy-an-azure-kubernetes-service-aks-cluster-using-an-arm-template"></a>Inicio rápido: Implementación de un clúster de Azure Kubernetes Service (AKS) mediante una plantilla de ARM

Azure Kubernetes Service (AKS) es un servicio de Kubernetes administrado que le permite implementar y administrar clústeres rápidamente. En este inicio rápido realizará lo siguiente:
* Implementación de un clúster de AKS mediante una plantilla de Azure Resource Manager. 
* Ejecución de una aplicación de varios contenedores con un servidor front-end web y una instancia de Redis en el clúster. 

![Imagen de la exploración hasta Azure Vote](media/container-service-kubernetes-walkthrough/azure-voting-application.png)

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

En esta guía rápida se presupone un conocimiento básico de los conceptos de Kubernetes. Para más información, consulte [Conceptos básicos de Kubernetes de Azure Kubernetes Service (AKS)][kubernetes-concepts].

Si su entorno cumple los requisitos previos y está familiarizado con el uso de plantillas de Resource Manager, seleccione el botón **Implementar en Azure**. La plantilla se abrirá en Azure Portal.

[![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.kubernetes%2Faks%2Fazuredeploy.json)

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- En este artículo se necesita la versión 2.0.61 de la CLI de Azure, o cualquier versión posterior. Si usa Azure Cloud Shell, ya está instalada la versión más reciente.

- Para crear un clúster de AKS con una plantilla de Resource Manager, proporcione una clave pública SSH. Si necesita este recurso, consulte la sección siguiente; en caso contrario, vaya a la sección [Revisión de la plantilla](#review-the-template).

### <a name="create-an-ssh-key-pair"></a>Creación de un par de claves SSH

Para acceder a los nodos de AKS, se conectará mediante un par de claves SSH (pública y privada), que se genera mediante el comando `ssh-keygen`. De forma predeterminada, estos archivos se crean en el directorio *~/.ssh*. La ejecución del comando `ssh-keygen` sobrescribirá cualquier par de claves SSH con el mismo nombre que ya exista en la ubicación especificada.

1. Vaya a [https://shell.azure.com](https://shell.azure.com) para abrir Cloud Shell en el explorador.

1. Ejecute el comando `ssh-keygen`. El siguiente comando crea un par de claves SSH con ayuda del cifrado RSA y una longitud en bits de 2048:

    ```console
    ssh-keygen -t rsa -b 2048
    ```

Para más información sobre cómo crear claves SSH, consulte el artículo sobre [Creación y administración de claves SSH para la autenticación en Azure][ssh-keys].

## <a name="review-the-template"></a>Revisión de la plantilla

La plantilla usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/aks/).

:::code language="json" source="~/quickstart-templates/quickstarts/microsoft.kubernetes/aks/azuredeploy.json":::

Para más ejemplos de AKS, consulte el sitio de [plantillas de inicio rápido de AKS][aks-quickstart-templates].

## <a name="deploy-the-template"></a>Implementación de la plantilla

1. Seleccione el botón siguiente para iniciar sesión en Azure y abrir una plantilla.

    [![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.kubernetes%2Faks%2Fazuredeploy.json)

2. Seleccione o escriba los siguientes valores.

    Para este inicio rápido, deje los valores predeterminados de *Tamaño del disco del SO GB*, *Número de agentes*, *Tamaño de VM del agente*, *Tipo de SO* y *Versión de Kubernetes*. Proporcione sus propios valores para los siguientes parámetros de plantilla:

    * **Suscripción**: Seleccione una suscripción de Azure.
    * **Grupo de recursos**: Seleccione **Crear nuevo**. Escriba un nombre único para el grupo de recursos, como *myResourceGroup*, y elija **Aceptar**.
    * **Ubicación**: seleccione una ubicación, como **Este de EE. UU**.
    * **Nombre del clúster**: escriba un nombre único para el clúster de AKS, como *myAKSCluster*.
    * **Prefijo de DNS**: escriba un prefijo DNS único para el clúster, como *myakscluster*.
    * **Linux Admin Username** (Nombre de usuario administrador de Linux): escriba un nombre de usuario para conectarse mediante SSH, como *azureuser*.
    * **SSH RSA Public Key** (Clave pública SSH RSA): copie y pegue la parte *pública* del par de claves SSH (de forma predeterminada, el contenido de *~/.ssh/id_rsa.pub*).

    ![Plantilla de Resource Manager para la creación de un clúster de Azure Kubernetes Service en el portal](./media/kubernetes-walkthrough-rm-template/create-aks-cluster-using-template-portal.png)

3. Seleccione **Revisar + crear**.

El clúster de AKS tarda unos minutos en crearse. Espere a que el clúster se implemente correctamente para pasar al siguiente paso.

## <a name="validate-the-deployment"></a>Validación de la implementación

### <a name="connect-to-the-cluster"></a>Conectarse al clúster

Para administrar un clúster de Kubernetes, use [kubectl][kubectl], el cliente de línea de comandos de Kubernetes. Si usa Azure Cloud Shell, `kubectl` ya está instalado. 

1. Instale `kubectl` localmente mediante el comando [az aks install-cli][az-aks-install-cli]:

    ```azurecli
    az aks install-cli
    ```

2. Para configurar `kubectl` para conectarse a su clúster de Kubernetes, use el comando [az aks get-credentials][az-aks-get-credentials]. Con este comando se descargan las credenciales y se configura la CLI de Kubernetes para usarlas.

    ```azurecli-interactive
    az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
    ```

3. Compruebe la conexión al clúster con el comando [kubectl get][kubectl-get]. Este comando devuelve una lista de los nodos del clúster.

    ```console
    kubectl get nodes
    ```

    La salida muestra los nodos creados en los pasos anteriores. Asegúrese de que el estado de los nodos sea *Listo*:

    ```output
    NAME                       STATUS   ROLES   AGE     VERSION
    aks-agentpool-41324942-0   Ready    agent   6m44s   v1.12.6    
    aks-agentpool-41324942-1   Ready    agent   6m46s   v1.12.6
    aks-agentpool-41324942-2   Ready    agent   6m45s   v1.12.6
    ```

### <a name="run-the-application"></a>Ejecución de la aplicación

Un [archivo de manifiesto de Kubernetes][kubernetes-deployment] define el estado deseado del clúster, por ejemplo, qué imágenes de contenedor se van a ejecutar. 

En este inicio rápido se usa un manifiesto para crear todos los objetos necesarios para ejecutar la [aplicación Azure Vote][azure-vote-app]. Este manifiesto incluye dos [implementaciones de Kubernetes][kubernetes-deployment]:
* Las aplicaciones de Python de ejemplo de Azure Vote.
* Una instancia de Redis. 

También se crean dos [servicios de Kubernetes][kubernetes-service]:
* Un servicio interno para la instancia de Redis.
* Un servicio externo para acceder a la aplicación Azure Vote desde Internet.

1. Cree un archivo llamado `azure-vote.yaml`.
    * Si usa Azure Cloud Shell, este archivo se puede crear mediante `vi` o `nano` como si trabajara en un sistema físico o virtual.
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

### <a name="test-the-application"></a>Prueba de la aplicación

Cuando se ejecuta la aplicación, un servicio de Kubernetes expone el front-end de la aplicación a Internet. Este proceso puede tardar unos minutos en completarse.

Para supervisar el progreso, utilice el comando [kubectl get service][kubectl-get] con el argumento `--watch`.

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

![Imagen de la exploración hasta Azure Vote](media/container-service-kubernetes-walkthrough/azure-voting-application.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

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
[azure-dev-spaces]: ../dev-spaces/index.yml
[aks-quickstart-templates]: https://azure.microsoft.com/resources/templates/?term=Azure+Kubernetes+Service

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
[azure-cli-install]: /cli/azure/install-azure-cli
[sp-delete]: kubernetes-service-principal.md#additional-considerations
[azure-portal]: https://portal.azure.com
[kubernetes-deployment]: concepts-clusters-workloads.md#deployments-and-yaml-manifests
[kubernetes-service]: concepts-network.md#services
[kubernetes-dashboard]: kubernetes-dashboard.md
[ssh-keys]: ../virtual-machines/linux/create-ssh-keys-detailed.md
[az-ad-sp-create-for-rbac]: /cli/azure/ad/sp#az_ad_sp_create_for_rbac
