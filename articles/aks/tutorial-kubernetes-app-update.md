---
title: 'Tutorial de Kubernetes en Azure: Actualización de una aplicación'
description: En este tutorial de Azure Kubernetes Service (AKS), aprenderá a actualizar la implementación de una aplicación existente a AKS con una nueva versión del código de la aplicación.
services: container-service
ms.topic: tutorial
ms.date: 05/24/2021
ms.custom: mvc, devx-track-azurepowershell
ms.openlocfilehash: eaa7f6b0f99a856ea9210be3fdb6d2de1dd87d2a
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110697955"
---
# <a name="tutorial-update-an-application-in-azure-kubernetes-service-aks"></a>Tutorial: Actualización de una aplicación en Azure Kubernetes Service (AKS)

Después de implementar una aplicación en Kubernetes, se puede actualizar especificando una nueva imagen de contenedor o la versión de la imagen. Una actualización se preconfigura para que solo una parte de la implementación se actualice al mismo tiempo. Esta actualización preconfigurada permite que la aplicación siga ejecutándose durante la actualización. También proporciona un mecanismo de reversión si se produce un error de implementación.

En este tutorial, la sección seis de siete, se actualiza la aplicación de ejemplo de Azure Vote. Aprenderá a:

> [!div class="checklist"]
> * Actualizar el código de la aplicación de front-end
> * Crear una imagen de contenedor actualizada
> * Insertar una imagen de contenedor en Azure Container Registry
> * Implementar la imagen de contenedor actualizada

## <a name="before-you-begin"></a>Antes de empezar

En los tutoriales anteriores se empaquetó una aplicación en una imagen de contenedor. Esta imagen se cargó en Azure Container Registry y el usuario creó un clúster de AKS. Luego la aplicación se implementó en el clúster de AKS.

También se clonó un repositorio de aplicaciones que incluye el código fuente de la aplicación y un archivo de Docker Compose creado previamente que se usa en este tutorial. Confirme que ha creado un clon del repositorio y que ha cambiado los directorios en el repositorio clonado. Si no ha finalizado estos pasos y desea continuar, comience con el [Tutorial 1: Creación de imágenes de contenedor][aks-tutorial-prepare-app].

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para realizar este tutorial es necesario ejecutar la versión 2.0.53, o superior, de la CLI de Azure. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure][azure-cli-install].

### <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

Para realizar este tutorial es preciso ejecutar la versión 5.9.0 de Azure PowerShell, o cualquier versión posterior. Ejecute `Get-InstalledModule -Name Az` para encontrar la versión. Si necesita instalarla o actualizarla, consulte el artículo sobre la [instalación de Azure PowerShell][azure-powershell-install].

---

## <a name="update-an-application"></a>Actualización de una aplicación

Vamos a realizar un cambio en la aplicación de ejemplo y, después, a actualizar la versión ya implementada en el clúster de AKS. Asegúrese de que está en el directorio clonado *azure-voting-app-redis*. El código fuente de la aplicación de ejemplo se puede encontrar en el directorio *azure-vote*. Abra el archivo *config_file.cfg* con un editor, como `vi`:

```console
vi azure-vote/azure-vote/config_file.cfg
```

Cambie los valores de *VOTE1VALUE* y *VOTE2VALUE* por otros diferentes, como colores. En el ejemplo siguiente se muestran los valores actualizados:

```
# UI Configurations
TITLE = 'Azure Voting App'
VOTE1VALUE = 'Blue'
VOTE2VALUE = 'Purple'
SHOWHOST = 'false'
```

Guarde y cierre el archivo. En `vi`, use `:wq`.

## <a name="update-the-container-image"></a>Actualización de la imagen del contenedor

Para volver a crear la imagen del front-end y probar la aplicación actualizada, use [docker-compose][docker-compose]. El argumento `--build` se emplea para indicar a Docker Compose que vuelva a crear la imagen de la aplicación.

```console
docker-compose up --build -d
```

## <a name="test-the-application-locally"></a>Prueba de la aplicación de forma local

Para comprobar que la imagen de contenedor actualizada muestra los cambios, abra un explorador web local en `http://localhost:8080`.

:::image type="content" source="media/container-service-kubernetes-tutorials/vote-app-updated.png" alt-text="Captura de pantalla que muestra un ejemplo de la imagen de contenedor actualizada de Azure Voting App, que se ejecuta localmente, abierta en un explorador web local.":::

Los valores actualizados que se proporcionan en el archivo *config_file.cfg* se muestran en la aplicación en ejecución.

## <a name="tag-and-push-the-image"></a>Etiquetado e inserción de la imagen

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para usar correctamente la imagen actualizada, etiquete la imagen *azure-vote-front* con el nombre del servidor de inicio de sesión de seguridad del registro de ACR. Para obtener el nombre del servidor de inicio de sesión, use el comando [az acr list](/cli/azure/acr):

```azurecli
az acr list --resource-group myResourceGroup --query "[].{acrLoginServer:loginServer}" --output table
```

### <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

Para usar correctamente la imagen actualizada, etiquete la imagen *azure-vote-front* con el nombre del servidor de inicio de sesión de seguridad del registro de ACR. Obtenga el nombre del servidor de inicio de sesión con el cmdlet [Get-AzContainerRegistry][get-azcontainerregistry]:

```azurepowershell
(Get-AzContainerRegistry -ResourceGroupName myResourceGroup -Name <acrName>).LoginServer
```

---


Use la [docker tag][docker-tag] para etiquetar la imagen. Reemplace `<acrLoginServer>` por el nombre del servidor de inicio de sesión de ACR o el nombre de host del registro público y actualice la versión de la imagen a *: v2* como se muestra a continuación:

```console
docker tag mcr.microsoft.com/azuredocs/azure-vote-front:v1 <acrLoginServer>/azure-vote-front:v2
```

Ahora use [docker push][docker-push] para cargar la imagen en el registro. Reemplace `<acrLoginServer>` por el nombre del servidor de inicio de sesión de ACR.

### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

> [!NOTE]
> Si surgen problemas al realizar inserciones en el registro de ACR, asegúrese de que no se ha cerrado la sesión. Ejecute el comando [az acr login][az-acr-login] y use el nombre de la instancia de Azure Container Registry que creó en el paso [Creación de una instancia de Azure Container Registry](tutorial-kubernetes-prepare-acr.md#create-an-azure-container-registry). Por ejemplo, `az acr login --name <azure container registry name>`.

### <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

> [!NOTE]
> Si surgen problemas al realizar inserciones en el registro de ACR, asegúrese de que no se ha cerrado la sesión. Ejecute el comando [Connect-AzContainerRegistry][connect-azcontainerregistry] y use el nombre de la instancia de Azure Container Registry que creó en el paso [Creación de una instancia de Azure Container Registry](tutorial-kubernetes-prepare-acr.md#create-an-azure-container-registry). Por ejemplo, `Connect-AzContainerRegistry -Name <azure container registry name>`.

---

```console
docker push <acrLoginServer>/azure-vote-front:v2
```

## <a name="deploy-the-updated-application"></a>Implementación de la aplicación actualizada

Para proporcionar el tiempo máximo de actividad, se deben ejecutar varias instancias del pod de aplicación. Compruebe el número de instancias de front-end en ejecución con el comando [kubectl get pods][kubectl-get]:

```
$ kubectl get pods

NAME                               READY     STATUS    RESTARTS   AGE
azure-vote-back-217588096-5w632    1/1       Running   0          10m
azure-vote-front-233282510-b5pkz   1/1       Running   0          10m
azure-vote-front-233282510-dhrtr   1/1       Running   0          10m
azure-vote-front-233282510-pqbfk   1/1       Running   0          10m
```

Si no tiene varios pods de front-end, escale la implementación de *azure-vote-front* de la manera siguiente:

```console
kubectl scale --replicas=3 deployment/azure-vote-front
```

Use el comando [kubectl set][kubectl-set] para actualizar la aplicación. Actualice `<acrLoginServer>` con el nombre de host o de servidor de inicio de sesión del registro del contenedor y especifique la versión de la aplicación *v2*:

```console
kubectl set image deployment azure-vote-front azure-vote-front=<acrLoginServer>/azure-vote-front:v2
```

Para supervisar la implementación, use el comando [kubectl get pod][kubectl-get]. A medida que se implementa la aplicación actualizada, sus pods se terminan y se vuelven a crear con la nueva imagen de contenedor.

```console
kubectl get pods
```

La salida del ejemplo siguiente muestra los pods que terminan y las nuevas instancias que se ejecutan a medida que progresa la implementación:

```
$ kubectl get pods

NAME                               READY     STATUS        RESTARTS   AGE
azure-vote-back-2978095810-gq9g0   1/1       Running       0          5m
azure-vote-front-1297194256-tpjlg  1/1       Running       0          1m
azure-vote-front-1297194256-tptnx  1/1       Running       0          5m
azure-vote-front-1297194256-zktw9  1/1       Terminating   0          1m
```

## <a name="test-the-updated-application"></a>Prueba de la aplicación actualizada

Para ver la aplicación actualizada, obtenga primero la dirección IP externa del servicio `azure-vote-front`:

```console
kubectl get service azure-vote-front
```

Ahora, abra un explorador web a la dirección IP del servicio:

:::image type="content" source="media/container-service-kubernetes-tutorials/vote-app-updated-external.png" alt-text="Captura de pantalla que muestra un ejemplo de la imagen actualizada de Azure Voting App, que se ejecuta en un clúster de AKS, abierta en un explorador web local.":::

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha actualizado una aplicación y ha implementado esta actualización en el clúster de AKS. Ha aprendido a:

> [!div class="checklist"]
> * Actualizar el código de la aplicación de front-end
> * Crear una imagen de contenedor actualizada
> * Insertar una imagen de contenedor en Azure Container Registry
> * Implementar la imagen de contenedor actualizada

Pase al siguiente tutorial para aprender a actualizar un clúster de AKS a una nueva versión de Kubernetes.

> [!div class="nextstepaction"]
> [Actualización de Kubernetes][aks-tutorial-upgrade]

<!-- LINKS - external -->
[docker-compose]: https://docs.docker.com/compose/
[docker-push]: https://docs.docker.com/engine/reference/commandline/push/
[docker-tag]: https://docs.docker.com/engine/reference/commandline/tag/
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubectl-set]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#set

<!-- LINKS - internal -->
[aks-tutorial-prepare-app]: ./tutorial-kubernetes-prepare-app.md
[aks-tutorial-upgrade]: ./tutorial-kubernetes-upgrade-cluster.md
[az-acr-login]: /cli/azure/acr
[azure-cli-install]: /cli/azure/install-azure-cli
[azure-powershell-install]: /powershell/azure/install-az-ps
[get-azcontainerregistry]: /powershell/module/az.containerregistry/get-azcontainerregistry
[connect-azcontainerregistry]: /powershell/module/az.containerregistry/connect-azcontainerregistry
