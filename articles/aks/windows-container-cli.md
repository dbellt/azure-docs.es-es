---
title: Creación de un contenedor de Windows Server en un clúster de AKS mediante la CLI de Azure
description: Aprenda a crear rápidamente un clúster de Kubernetes y a implementar una aplicación en un contenedor de Windows Server en Azure Kubernetes Service (AKS) mediante la CLI de Azure.
services: container-service
ms.topic: article
ms.date: 07/16/2020
ms.openlocfilehash: 50b5d0a46c97cfd816b80c3fb7c8f8667e3e89d7
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110379377"
---
# <a name="create-a-windows-server-container-on-an-azure-kubernetes-service-aks-cluster-using-the-azure-cli"></a>Creación de un contenedor de Windows Server en un clúster de Azure Kubernetes Service (AKS) mediante la CLI de Azure

Azure Kubernetes Service (AKS) es un servicio de Kubernetes administrado que le permite implementar y administrar clústeres rápidamente. En este artículo, implementará un clúster de AKS mediante la CLI de Azure. También implementará una aplicación de ejemplo de ASP.NET en un contenedor de Windows Server en el clúster.

![Imagen de cómo llegar a la aplicación de ejemplo de ASP.NET](media/windows-container/asp-net-sample-app.png)

En este artículo se presupone un conocimiento básico de los conceptos de Kubernetes. Para más información, consulte [Conceptos básicos de Kubernetes de Azure Kubernetes Service (AKS)][kubernetes-concepts].

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

### <a name="limitations"></a>Limitaciones

Se aplican las siguientes limitaciones cuando crea y administra clústeres de AKS que admiten varios grupos de nodos:

* No puede eliminar el primer grupo de nodos.

Se aplican las siguientes limitaciones adicionales a los grupos de nodos de Windows Server:

* El clúster de AKS puede tener un máximo de 10 grupos de nodos.
* El clúster de AKS puede tener un máximo de 100 nodos en cada grupo de nodos.
* El nombre del grupo de nodos de Windows Server tiene un límite de 6 caracteres.

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Un grupo de recursos de Azure es un grupo lógico en el que se implementan y administran recursos de Azure. Cuando se crea un grupo de recursos, se le pide que especifique una ubicación. Dicha ubicación es donde se almacenan los metadatos del grupo de recursos, así como el lugar en el que los recursos se ejecutan en Azure si no se especifica otra región al crear los recursos. Cree un grupo de recursos con el comando [az group create][az-group-create].

En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroup* en la ubicación *eastus*.

> [!NOTE]
> En este artículo se utiliza sintaxis de Bash para los comandos del tutorial.
> Si usa Azure Cloud Shell, asegúrese de que la lista desplegable de la superior izquierda de la ventana de Cloud Shell está establecida en **Bash**.

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

En la siguiente salida de ejemplo se muestra que los recursos se crearon correctamente:

```json
{
  "id": "/subscriptions/<guid>/resourceGroups/myResourceGroup",
  "location": "eastus",
  "managedBy": null,
  "name": "myResourceGroup",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null,
  "type": null
}
```

## <a name="create-an-aks-cluster"></a>Creación de un clúster de AKS

Para ejecutar un clúster de AKS que admita grupos de nodos para contenedores de Windows Server, el clúster debe utilizar una directiva de red que use el complemento de red (avanzado) de [Azure CNI][azure-cni-about]. Para más información que le ayude a planear los intervalos de subred necesarios y las consideraciones de red, vea [Configuración de redes de Azure CNI][use-advanced-networking]. Use el comando [az aks create][az-aks-create] para crear un clúster de AKS denominado *myAKSCluster*. Este comando creará los recursos de red necesarios en caso de que no existan.

* El clúster se configura con dos nodos.
* Los parámetros `--windows-admin-password` y `--windows-admin-username` establecen las credenciales de administrador para los nodos de Windows Server del clúster y deben satisfacer los [requisitos de contraseña de Windows Server][windows-server-password]. Si no especifica el parámetro *windows-admin-password*, se le pedirá que proporcione un valor.
* El grupo de nodos usa `VirtualMachineScaleSets`.

> [!NOTE]
> Para asegurarse de que el clúster funcione de forma confiable, debe ejecutar al menos 2 (dos) nodos del grupo de nodos predeterminado.

Cree un nombre de usuario para usarlo como credenciales de administrador para los nodos de Windows Server en el clúster. Los comandos siguientes le solicitan un nombre de usuario y lo establecen en WINDOWS_USERNAME para su uso en un comando posterior (recuerde que los comandos de este artículo se incluyen en un shell de BASH).

```azurecli-interactive
echo "Please enter the username to use as administrator credentials for Windows Server nodes on your cluster: " && read WINDOWS_USERNAME
```

Cree el clúster y asegúrese de especificar el parámetro `--windows-admin-username`. El siguiente comando de ejemplo crea un clúster con el valor de *WINDOWS_USERNAME* establecido en el comando anterior. También, puede proporcionar un nombre de usuario diferente directamente en el parámetro, en lugar de usar *WINDOWS_USERNAME*. El siguiente comando también le pedirá que cree una contraseña para las credenciales de administrador de los nodos de Windows Server en el clúster. Como alternativa, puede usar el parámetro *windows-admin-password* y especificar aquí su propio valor.

```azurecli-interactive
az aks create \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --node-count 2 \
    --enable-addons monitoring \
    --generate-ssh-keys \
    --windows-admin-username $WINDOWS_USERNAME \
    --vm-set-type VirtualMachineScaleSets \
    --kubernetes-version 1.20.2 \
    --network-plugin azure
```

> [!NOTE]
> Si recibe un error de validación de contraseña, compruebe que el parámetro establecido cumple los [requisitos de contraseña de Windows Server][windows-server-password]. Si la contraseña cumple los requisitos, pruebe a crear el grupo de recursos en otra región. A continuación, intente crear el clúster con el nuevo grupo de recursos.
>
> Si no especifica un nombre de usuario y una contraseña de administrador al establecer `--vm-set-type VirtualMachineScaleSets` y `--network-plugin azure`, el nombre de usuario se establece en *azureuser* y la contraseña se establece en un valor aleatorio.
> 
> No se puede cambiar el nombre de usuario del administrador, pero puede cambiar la contraseña de administrador que el clúster de AKS usa para los nodos de Windows Server mediante `az aks update`. Para más información, consulte [Preguntas frecuentes sobre los grupos de nodos de Windows Server][win-faq-change-admin-creds].

Transcurridos unos minutos, el comando se completa y devuelve información en formato JSON sobre el clúster. En ocasiones, el clúster puede tardar más de unos minutos en aprovisionarse. Espere hasta 10 minutos en estos casos.

## <a name="add-a-windows-server-node-pool"></a>Adición de un grupo de nodos de Windows Server

De forma predeterminada, se crea un clúster de AKS con un grupo de nodos que puede ejecutar contenedores de Linux. Utilice el comando `az aks nodepool add` para agregar un grupo de nodos adicionales que pueda ejecutar contenedores de Windows Server en combinación con el grupo de nodos de Linux.

```azurecli
az aks nodepool add \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --os-type Windows \
    --name npwin \
    --node-count 1
```

El comando anterior crea un nuevo grupo de nodos denominado *npwin* y lo agrega a *myAKSCluster*. El comando anterior también usa la subred predeterminada en la red virtual predeterminada que se crea al ejecutar `az aks create`.

### <a name="add-a-windows-server-node-pool-with-containerd-preview"></a>Adición de un grupo de nodos de Windows Server con `containerd` (versión preliminar)

A partir de la versión 1.20 de Kubernetes y versiones posteriores, puede especificar `containerd` como el entorno de ejecución del contenedor para los grupos de nodos de Windows Server 2019.

[!INCLUDE [preview features callout](./includes/preview/preview-callout.md)]

Necesitará la extensión de la CLI de Azure *aks-preview*. Instale la extensión de la CLI de Azure *aks-preview* mediante el comando [az extension add][az-extension-add]. También puede instalar las actualizaciones disponibles mediante el comando [az extension update][az-extension-update].

```azurecli-interactive
# Install the aks-preview extension
az extension add --name aks-preview

# Update the extension to make sure you have the latest version installed
az extension update --name aks-preview
```

Registro de `UseCustomizedWindowsContainerRuntime` la marca de característica con el comando de [característica de registro az][az-feature-register], tal como se muestra en el siguiente ejemplo:

```azurecli
az feature register --namespace "Microsoft.ContainerService" --name "UseCustomizedWindowsContainerRuntime"
```

Puede comprobar el estado de registro con el comando [az feature list][az-feature-list]:

```azurecli
az feature list -o table --query "[?contains(name, 'Microsoft.ContainerService/UseCustomizedWindowsContainerRuntime')].{Name:name,State:properties.state}"
```

Cuando todo esté listo, actualice el registro del proveedor de recursos Microsoft.ContainerService con el comando [az provider register][az-provider-register]:

```azurecli
az provider register --namespace Microsoft.ContainerService
```

Use el comando `az aks nodepool add` para agregar un grupo de nodos adicionales que pueda ejecutar contenedores de Windows Server con el entorno de ejecución `containerd`.

> [!NOTE]
> Si no especifica el encabezado personalizado *WindowsContainerRuntime=containerd*, el grupo de nodos usará Docker como entorno de ejecución de contenedor.

```azurecli
az aks nodepool add \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --os-type Windows \
    --name npwcd \
    --node-vm-size Standard_D4s_v3 \
    --kubernetes-version 1.20.2 \
    --aks-custom-headers WindowsContainerRuntime=containerd \
    --node-count 1
```

El comando anterior crea un nuevo grupo de nodos de Windows Server que usa `containerd` como entorno de ejecución, llamado *npwcd*, y lo agrega a *myAKSCluster*. El comando anterior también usa la subred predeterminada en la red virtual predeterminada que se crea al ejecutar `az aks create`.

> [!IMPORTANT]
> Cuando se utiliza `containerd` con grupos de nodos de Windows Server 2019:
> - Tanto el plano de control como los grupos de nodos de Windows Server 2019 deben usar la versión 1.20 o posterior de Kubernetes.
> - Los grupos de nodos existentes de Windows Server 2019 que utilizan Docker como entorno de ejecución de contenedor no se pueden actualizar para usar `containerd`. Debe crear un nuevo grupo de nodos.
> - Al crear un grupo de nodos para ejecutar contenedores de Windows Server, el valor predeterminado de *node-vm-size* es *Standard_D2s_v3* que era el tamaño mínimo recomendado para los grupos de nodos de Windows Server 2019 en las versiones anteriores a Kubernetes 1.20. El tamaño mínimo recomendado para grupos de nodos de Windows Server 2019 cuando se usa `containerd` es *Standard_D4s_v3*. Si establece el parámetro *node-vm-size*, compruebe la lista de [tamaños de máquina virtual restringidos][restricted-vm-sizes].
> - Se recomienda encarecidamente usar valores [taints o etiquetas][aks-taints] con los grupos de nodos de Windows Server 2019 en ejecución `containerd` y tolerations o selectores de nodos con las implementaciones, para garantizar que las cargas de trabajo se programan correctamente.

## <a name="connect-to-the-cluster"></a>Conectarse al clúster

Para administrar un clúster de Kubernetes, usará [kubectl][kubectl], el cliente de línea de comandos de Kubernetes. Si usa Azure Cloud Shell, `kubectl` ya está instalado. Para instalar `kubectl` localmente, use el comando [az aks install-cli][az-aks-install-cli]:

```azurecli
az aks install-cli
```

Para configurar `kubectl` para conectarse a su clúster de Kubernetes, use el comando [az aks get-credentials][az-aks-get-credentials]. Con este comando se descargan las credenciales y se configura la CLI de Kubernetes para usarlas.

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
```

Para comprobar la conexión al clúster, use el comando [kubectl get][kubectl-get] para devolver una lista de los nodos del clúster.

```console
kubectl get nodes -o wide
```

La siguiente salida de ejemplo muestra todos los nodos del clúster. Asegúrese de que el estado de todos los nodos sea *Listo*:

```output
NAME                                STATUS   ROLES   AGE    VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                         KERNEL-VERSION     CONTAINER-RUNTIME
aks-nodepool1-12345678-vmss000000   Ready    agent   34m    v1.20.2   10.240.0.4    <none>        Ubuntu 18.04.5 LTS               5.4.0-1046-azure   containerd://1.4.4+azure
aks-nodepool1-12345678-vmss000001   Ready    agent   34m    v1.20.2   10.240.0.35   <none>        Ubuntu 18.04.5 LTS               5.4.0-1046-azure   containerd://1.4.4+azure
aksnpwcd123456                      Ready    agent   9m6s   v1.20.2   10.240.0.97   <none>        Windows Server 2019 Datacenter   10.0.17763.1879    containerd://1.4.4+unknown
aksnpwin987654                      Ready    agent   25m    v1.20.2   10.240.0.66   <none>        Windows Server 2019 Datacenter   10.0.17763.1879    docker://19.3.14
```

> [!NOTE]
> El entorno de ejecución de contenedor para cada grupo de nodos se muestra en *CONTAINER-RUNTIME*. Observe que *aksnpwin987654* comienza por `docker://`, lo que significa que usa Docker para el entorno de ejecución de contenedor. Observe que *aksnpwcd123456* comienza por `containerd://`, lo que significa que usa `containerd` para el entorno de ejecución de contenedor.

## <a name="run-the-application"></a>Ejecución de la aplicación

Un archivo de manifiesto de Kubernetes define un estado deseado del clúster, por ejemplo, qué imágenes de contenedor se van a ejecutar. En este artículo, se utiliza un manifiesto para crear todos los objetos necesarios para ejecutar la aplicación de ejemplo de ASP.NET en un contenedor de Windows Server. Este manifiesto incluye una [implementación de Kubernetes][kubernetes-deployment] para la aplicación de ejemplo de ASP.NET y un [servicio de Kubernetes][kubernetes-service] externo para acceder a la aplicación desde Internet.

La aplicación de ejemplo de ASP.NET se proporciona como parte de los [ejemplos de .NET Framework][dotnet-samples] y se ejecuta en un contenedor de Windows Server. AKS requiere contenedores de Windows Server que se basen en las imágenes de *Windows Server 2019* u otra versión posterior. El archivo de manifiesto de Kubernetes también debe definir un [selector de nodos][node-selector] que le indique al clúster de AKS que ejecute el pod de la aplicación de ejemplo de ASP.NET en un nodo que pueda ejecutar contenedores de Windows Server.

Cree un archivo denominado `sample.yaml` y cópielo en la siguiente definición de código YAML. Si usa Azure Cloud Shell, este archivo se puede crear mediante `vi` o `nano` como si funcionara en un sistema físico o virtual:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample
  labels:
    app: sample
spec:
  replicas: 1
  template:
    metadata:
      name: sample
      labels:
        app: sample
    spec:
      nodeSelector:
        "beta.kubernetes.io/os": windows
      containers:
      - name: sample
        image: mcr.microsoft.com/dotnet/framework/samples:aspnetapp
        resources:
          limits:
            cpu: 1
            memory: 800M
          requests:
            cpu: .1
            memory: 300M
        ports:
          - containerPort: 80
  selector:
    matchLabels:
      app: sample
---
apiVersion: v1
kind: Service
metadata:
  name: sample
spec:
  type: LoadBalancer
  ports:
  - protocol: TCP
    port: 80
  selector:
    app: sample
```

Implemente la aplicación mediante el comando [kubectl apply][kubectl-apply] y especifique el nombre del manifiesto de YAML:

```console
kubectl apply -f sample.yaml
```

En la salida de ejemplo siguiente se muestran las implementaciones y los servicios creados correctamente:

```output
deployment.apps/sample created
service/sample created
```

## <a name="test-the-application"></a>Prueba de la aplicación

Cuando se ejecuta la aplicación, un servicio de Kubernetes expone el front-end de la aplicación a Internet. Este proceso puede tardar unos minutos en completarse. En ocasiones, el servicio puede tardar más de unos minutos en aprovisionarse. Espere hasta 10 minutos en estos casos.

Para supervisar el progreso, utilice el comando [kubectl get service][kubectl-get] con el argumento `--watch`.

```console
kubectl get service sample --watch
```

En un primer momento, el parámetro *EXTERNAL-IP* del servicio de *ejemplo* se muestra como *pendiente*.

```output
NAME               TYPE           CLUSTER-IP   EXTERNAL-IP   PORT(S)        AGE
sample             LoadBalancer   10.0.37.27   <pending>     80:30572/TCP   6s
```

Cuando la dirección *EXTERNAL-IP* cambie de *pendiente* a una dirección IP pública real, use `CTRL-C` para detener el proceso de inspección de `kubectl`. En la salida del ejemplo siguiente se muestra una dirección IP pública válida asignada al servicio:

```output
sample  LoadBalancer   10.0.37.27   52.179.23.131   80:30572/TCP   2m
```

Para ver la aplicación de ejemplo en acción, abra un explorador web en la dirección IP externa del servicio.

![Imagen de cómo llegar a la aplicación de ejemplo de ASP.NET](media/windows-container/asp-net-sample-app.png)

> [!Note]
> Si se agota el tiempo de espera de conexión al intentar cargar la página, debe comprobar que la aplicación de ejemplo esté lista con el siguiente comando [kubectl get pods --watch]. A veces, el contenedor Windows no se habrá iniciado en el momento en que la dirección IP externa esté disponible.

## <a name="delete-cluster"></a>Eliminación de clúster

Cuando el clúster ya no se necesite, puede usar el comando [az group delete][az-group-delete] para eliminar el grupo de recursos, el servicio de contenedor y todos los recursos relacionados.

```azurecli-interactive
az group delete --name myResourceGroup --yes --no-wait
```

> [!NOTE]
> Cuando elimina el clúster, la entidad de servicio Azure Active Directory que utiliza el clúster de AKS no se quita. Para conocer los pasos que hay que realizar para quitar la entidad de servicio, consulte [Consideraciones principales y eliminación de AKS][sp-delete]. Si usó una identidad administrada, esta está administrada por la plataforma y no requiere que la quite.

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha implementado un clúster de Kubernetes y una aplicación de ejemplo de ASP.NET en un contenedor de Windows Server de este. [Acceda al panel web de Kubernetes][kubernetes-dashboard] del clúster que acaba de crear.

Para obtener más información sobre AKS y un ejemplo completo desde el código hasta la implementación, continúe con el tutorial del clúster de Kubernetes.

> [!div class="nextstepaction"]
> [Tutorial de AKS][aks-tutorial]

<!-- LINKS - external -->
[kubectl]: https://kubernetes.io/docs/user-guide/kubectl/
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[node-selector]: https://kubernetes.io/docs/concepts/configuration/assign-pod-node/
[dotnet-samples]: https://hub.docker.com/_/microsoft-dotnet-framework-samples/
[azure-cni]: https://github.com/Azure/azure-container-networking/blob/master/docs/cni.md

<!-- LINKS - internal -->
[kubernetes-concepts]: concepts-clusters-workloads.md
[aks-monitor]: ../azure-monitor/containers/container-insights-onboard.md
[aks-tutorial]: ./tutorial-kubernetes-prepare-app.md
[aks-taints]:  use-multiple-node-pools.md#specify-a-taint-label-or-tag-for-a-node-pool
[az-aks-browse]: /cli/azure/aks#az_aks_browse
[az-aks-create]: /cli/azure/aks#az_aks_create
[az-aks-get-credentials]: /cli/azure/aks#az_aks_get_credentials
[az-aks-install-cli]: /cli/azure/aks#az_aks_install_cli
[az-extension-add]: /cli/azure/extension#az_extension_add
[az-feature-list]: /cli/azure/feature#az_feature_list
[az-feature-register]: /cli/azure/feature#az_feature_register
[az-group-create]: /cli/azure/group#az_group_create
[az-group-delete]: /cli/azure/group#az_group_delete
[az-provider-register]: /cli/azure/provider#az_provider_register
[azure-cli-install]: /cli/azure/install-azure-cli
[azure-cni-about]: concepts-network.md#azure-cni-advanced-networking
[sp-delete]: kubernetes-service-principal.md#additional-considerations
[azure-portal]: https://portal.azure.com
[kubernetes-deployment]: concepts-clusters-workloads.md#deployments-and-yaml-manifests
[kubernetes-service]: concepts-network.md#services
[kubernetes-dashboard]: kubernetes-dashboard.md
[restricted-vm-sizes]: quotas-skus-regions.md#restricted-vm-sizes
[use-advanced-networking]: configure-azure-cni.md
[aks-support-policies]: support-policies.md
[aks-faq]: faq.md
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update
[windows-server-password]: /windows/security/threat-protection/security-policy-settings/password-must-meet-complexity-requirements#reference
[win-faq-change-admin-creds]: windows-faq.md#how-do-i-change-the-administrator-password-for-windows-server-nodes-on-my-cluster
