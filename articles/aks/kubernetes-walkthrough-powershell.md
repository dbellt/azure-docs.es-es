---
title: 'Inicio rápido: Implementación de un clúster de AKS mediante PowerShell'
description: Aprenda a crear rápidamente un clúster de Kubernetes, implementar una aplicación y supervisar el rendimiento en Azure Kubernetes Service (AKS) mediante PowerShell.
services: container-service
ms.topic: quickstart
ms.date: 03/15/2021
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 0f96ecd00b72f1853fe25b497283ea48ebdf55d7
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111747804"
---
# <a name="quickstart-deploy-an-azure-kubernetes-service-cluster-using-powershell"></a>Inicio rápido: Implementación de un clúster de Azure Kubernetes Service mediante PowerShell

Azure Kubernetes Service (AKS) es un servicio de Kubernetes administrado que le permite implementar y administrar clústeres rápidamente. En este inicio rápido realizará lo siguiente:
* Implementar un clúster de AKS mediante PowerShell.
* Ejecutar en el clúster una aplicación de varios contenedores con un front-end web y una instancia de Redis.
* Supervisar el mantenimiento del clúster y los pods que ejecutan la aplicación.

Para más información sobre cómo crear un grupo de nodos de Windows Server, consulte [Creación de un clúster de AKS que admita contenedores de Windows Server][windows-container-powershell].

![Aplicación de votación implementada en Azure Kubernetes Service](./media/kubernetes-walkthrough-powershell/voting-app-deployed-in-azure-kubernetes-service.png)

En esta guía rápida se presupone un conocimiento básico de los conceptos de Kubernetes. Para más información, consulte [Conceptos básicos de Kubernetes de Azure Kubernetes Service (AKS)][kubernetes-concepts].

## <a name="prerequisites"></a>Prerrequisitos

Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

Si va a ejecutar PowerShell de forma local, instale el módulo Az PowerShell y conéctese a su cuenta de Azure con el cmdlet [Connect-AzAccount](/powershell/module/az.accounts/Connect-AzAccount). Para más información sobre cómo instalar el módulo Az PowerShell, consulte [Instalación de Azure PowerShell][install-azure-powershell].

[!INCLUDE [cloud-shell-try-it](../../includes/cloud-shell-try-it.md)]

Si tiene varias suscripciones a Azure, seleccione el identificador de la suscripción adecuada en la que se deben facturar los recursos con el cmdlet [Set-AzContext](/powershell/module/az.accounts/set-azcontext).

```azurepowershell-interactive
Set-AzContext -SubscriptionId 00000000-0000-0000-0000-000000000000
```

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Un [grupo de recursos de Azure](../azure-resource-manager/management/overview.md) es un grupo lógico en el que se implementan y administran recursos de Azure. Cuando crea un grupo de recursos, se le pide que especifique una ubicación. Esta ubicación es:
* La ubicación de almacenamiento de los metadatos del grupo de recursos.
* El lugar en el que se ejecutan los recursos en Azure si no se especifica otra región al crear los recursos.

En el ejemplo siguiente, se crea un grupo de recursos denominado **myResourceGroup** en la región **eastus**.

Cree un grupo de recursos con el cmdlet [New-AzResourceGroup][new-azresourcegroup].

```azurepowershell-interactive
New-AzResourceGroup -Name myResourceGroup -Location eastus
```

Salida del grupo de recursos que se creó correctamente:

```plaintext
ResourceGroupName : myResourceGroup
Location          : eastus
ProvisioningState : Succeeded
Tags              :
ResourceId        : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup
```

## <a name="create-aks-cluster"></a>Creación de un clúster de AKS

1. Use la utilidad de línea de comandos `ssh-keygen` para generar un par de claves SSH.
    * Para más información, consulte [Pasos rápidos: Creación y uso de un par de claves pública-privada SSH para máquinas virtuales Linux en Azure](../virtual-machines/linux/mac-create-ssh-keys.md).

1. Use el cmdlet [New-AzAksCluster][new-azakscluster] para crear un clúster de AKS. Azure Monitor para contenedores está habilitado de forma predeterminada.

    En el siguiente ejemplo se crea un clúster denominado **myAKSCluster** con un nodo.

    ```azurepowershell-interactive
    New-AzAksCluster -ResourceGroupName myResourceGroup -Name myAKSCluster -NodeCount 1
    ```

Transcurridos unos minutos, el comando se completa y devuelve información en formato JSON sobre el clúster.

> [!NOTE]
> Al crear un clúster de AKS, se crea automáticamente un segundo grupo de recursos para almacenar los recursos de AKS. Para más información, consulte [¿Por qué se crean dos grupos de recursos con AKS?](./faq.md#why-are-two-resource-groups-created-with-aks)

## <a name="connect-to-the-cluster"></a>Conectarse al clúster

Para administrar un clúster de Kubernetes, use [kubectl][kubectl], el cliente de línea de comandos de Kubernetes. Si usa Azure Cloud Shell, `kubectl` ya está instalado.

1. Para instalar `kubectl` localmente, use el cmdlet `Install-AzAksKubectl`:

    ```azurepowershell
    Install-AzAksKubectl
    ```

2. Para configurar `kubectl` para conectarse al clúster de Kubernetes, use el cmdlet [Import-AzAksCredential][import-azakscredential]. El siguiente cmdlet permite descargar las credenciales y configurar la CLI de Kubernetes para usarlas.

    ```azurepowershell-interactive
    Import-AzAksCredential -ResourceGroupName myResourceGroup -Name myAKSCluster
    ```

3. Para comprobar la conexión al clúster, ejecute el comando [kubectl get][kubectl-get]. Este comando devuelve una lista de los nodos del clúster.

    ```azurepowershell-interactive
    .\kubectl get nodes
    ```

    La salida muestra el nodo único creado en los pasos anteriores. Asegúrese de que el estado del nodo es *Listo*:

    ```plaintext
    NAME                       STATUS   ROLES   AGE     VERSION
    aks-nodepool1-31718369-0   Ready    agent   6m44s   v1.15.10
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

    ```azurepowershell-interactive
    .\kubectl apply -f azure-vote.yaml
    ```

    La salida muestra los servicios y las implementaciones que se crearon correctamente:

    ```plaintext
    deployment.apps/azure-vote-back created
    service/azure-vote-back created
    deployment.apps/azure-vote-front created
    service/azure-vote-front created
    ```

## <a name="test-the-application"></a>Prueba de la aplicación

Cuando se ejecuta la aplicación, un servicio de Kubernetes expone el front-end de la aplicación a Internet. Este proceso puede tardar unos minutos en completarse.

Para supervisar el progreso, utilice el comando [kubectl get service][kubectl-get] con el argumento `--watch`.

```azurepowershell-interactive
.\kubectl get service azure-vote-front --watch
```

La salida de **EXTERNAL-IP** del servicio `azure-vote-front` aparecerá inicialmente como *pendiente*.

```plaintext
NAME               TYPE           CLUSTER-IP   EXTERNAL-IP   PORT(S)        AGE
azure-vote-front   LoadBalancer   10.0.37.27   <pending>     80:30572/TCP   6s
```

Una vez que la dirección **EXTERNAL-IP** cambia de *pendiente* a una dirección IP pública real, use `CTRL-C` para detener el `kubectl` proceso de inspección. En la salida del ejemplo siguiente se muestra una dirección IP pública válida asignada al servicio:

```plaintext
azure-vote-front   LoadBalancer   10.0.37.27   52.179.23.131   80:30572/TCP   2m
```

Para ver la aplicación Azure Vote en acción, abra un explorador web en la dirección IP externa del servicio.

![Aplicación de votación implementada en Azure Kubernetes Service](./media/kubernetes-walkthrough-powershell/voting-app-deployed-in-azure-kubernetes-service.png)

Vea las métricas de mantenimiento de los nodos del clúster y los pods capturadas por Azure Monitor para los contenedores de Azure Portal.

## <a name="delete-the-cluster"></a>Eliminación del clúster

Para evitar cargos de Azure, se recomienda limpiar los recursos que no sean necesarios. Use el cmdlet [Remove-AzResourceGroup][remove-azresourcegroup] para eliminar el grupo de recursos, el servicio de contenedor y todos los recursos relacionados.

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroup
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
[kubectl]: https://kubernetes.io/docs/user-guide/kubectl/
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[azure-dev-spaces]: ../dev-spaces/index.yml
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[azure-vote-app]: https://github.com/Azure-Samples/azure-voting-app-redis.git

<!-- LINKS - internal -->
[windows-container-powershell]: windows-container-powershell.md
[kubernetes-concepts]: concepts-clusters-workloads.md
[install-azure-powershell]: /powershell/azure/install-az-ps
[new-azresourcegroup]: /powershell/module/az.resources/new-azresourcegroup
[new-azakscluster]: /powershell/module/az.aks/new-azakscluster
[import-azakscredential]: /powershell/module/az.aks/import-azakscredential
[kubernetes-deployment]: concepts-clusters-workloads.md#deployments-and-yaml-manifests
[kubernetes-service]: concepts-network.md#services
[remove-azresourcegroup]: /powershell/module/az.resources/remove-azresourcegroup
[sp-delete]: kubernetes-service-principal.md#additional-considerations
[kubernetes-dashboard]: kubernetes-dashboard.md
[aks-tutorial]: ./tutorial-kubernetes-prepare-app.md
